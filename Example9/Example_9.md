# BlockChain-Sample
Block chained for engineers Example 9

## 9-1 Environment Setting

```
cd C:\Users\Sunghyun\AppData\Roaming\Ethereum Wallet\binaries\Geth\geth-windows-amd64-1.8.23-c9427004

geth --datadir home\eth_private_net init home\eth_private_net\genesis.json

geth --networkid "10" --nodiscover --datadir "home\eth_private_net" --rpc --rpcaddr "localhost" --rpcport "8545" --rpccorsdomain "*" --rpcapi "eth,net,web3,personal" --targetgaslimit "20000000" console 2>> home\eth_private_net\geth_err.log
```

## 9-2 Crowd fundding

```c
pragma solidity ^0.4.11;
contract CrowdFunding {
	// 투자자 구조체
	struct Investor {
		address addr;	// 투자자의 어드레스
		uint amount;	// 투자액
	}
	
	address public owner;		// 컨트랙트 소유자
	uint public numInvestors;	// 투자자 수
	uint public deadline;		// 마감일 (UnixTime)
	string public status;		// 모금활동 스테이터스
	bool public ended;			// 모금 종료여부
	uint public goalAmount;		// 목표액
	uint public totalAmount;	// 총 투자액
	mapping (uint => Investor) public investors;	// 투자자 관리를 위한 매핑
	
	modifier onlyOwner () {
		require(msg.sender == owner);
		_;
	}
	
	/// 생성자
	constructor(uint _duration, uint _goalAmount) public{
		owner = msg.sender;

		// 마감일 설정 (Unixtime)
		deadline = now + _duration;

		goalAmount = _goalAmount;
		status = "Funding";
		ended = false;

		numInvestors = 0;
		totalAmount = 0;
	}
	
	/// 투자 시에 호출되는 함수
	function fund() public payable {
		// 모금이 끝났다면 처리 중단
		require(!ended);
		
		Investor storage inv = investors[numInvestors++];
		inv.addr = msg.sender;
		inv.amount = msg.value;
		totalAmount += inv.amount;
	}
	
	/// 목표액 달성 여부 확인
	/// 그리고 모금 성공/실패 여부에 따라 송금
	function checkGoalReached () public onlyOwner {		
		// 모금이 끝났다면 처리 중단
		require(!ended);
		
		// 마감이 지나지 않았다면 처리 중단
		require(now >= deadline);
		
		if(totalAmount >= goalAmount) {	// 모금 성공인 경우
			status = "Campaign Succeeded";
			ended = true;
			// 컨트랙트 소유자에게 컨트랙트에 있는 모든 이더를 송금
			if(!owner.send(address(this).balance)) {
				revert();
			}
		} else {	// 모금 실패인 경우
			uint i = 0;
			status = "Campaign Failed";
			ended = true;
			
			// 각 투자자에게 투자금을 돌려줌
			while(i <= numInvestors) {
				if(!investors[i].addr.send(investors[i].amount)) {
					revert();
				}
				i++;
			}
		}
	}
	
	/// 컨트랙트를 소멸시키기 위한 함수
	function kill() public onlyOwner {
		selfdestruct(owner);
	}
}
```

### geth 콘솔로 옮겨가서 명령 실행하기

```
계약 정의 : var he = eth.contract(인터페이스_내용).at('어드레스')
이렇게 만들어진 계약을 'he'라는 변수를 통해 조작할 수 있다
```

### Interface ABI

```
var cf= eth.contract([ { "constant": false, "inputs": [], "name": "checkGoalReached", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0x01cb3b20" }, { "constant": true, "inputs": [], "name": "ended", "outputs": [ { "name": "", "type": "bool", "value": false } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x12fa6feb" }, { "constant": true, "inputs": [], "name": "numInvestors", "outputs": [ { "name": "", "type": "uint256", "value": "0" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x132ae5e9" }, { "constant": true, "inputs": [], "name": "totalAmount", "outputs": [ { "name": "", "type": "uint256", "value": "0" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x1a39d8ef" }, { "constant": true, "inputs": [], "name": "status", "outputs": [ { "name": "", "type": "string", "value": "Funding" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x200d2ed2" }, { "constant": true, "inputs": [], "name": "goalAmount", "outputs": [ { "name": "", "type": "uint256", "value": "10000000000" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x2636b945" }, { "constant": true, "inputs": [], "name": "deadline", "outputs": [ { "name": "", "type": "uint256", "value": "1557737266" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x29dcb0cf" }, { "constant": true, "inputs": [ { "name": "", "type": "uint256" } ], "name": "investors", "outputs": [ { "name": "addr", "type": "address", "value": "0x0000000000000000000000000000000000000000" }, { "name": "amount", "type": "uint256", "value": "0" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x3feb5f2b" }, { "constant": false, "inputs": [], "name": "kill", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0x41c0e1b5" }, { "constant": true, "inputs": [], "name": "owner", "outputs": [ { "name": "", "type": "address", "value": "0x92E735452E40569f21299138965dE184B67BD401" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x8da5cb5b" }, { "constant": false, "inputs": [], "name": "fund", "outputs": [], "payable": true, "stateMutability": "payable", "type": "function", "signature": "0xb60d4288" }, { "inputs": [ { "name": "_duration", "type": "uint256", "index": 0, "typeShort": "uint", "bits": "256", "displayName": "&thinsp;<span class=\"punctuation\">_</span>&thinsp;duration", "template": "elements_input_uint", "value": "600" }, { "name": "_goalAmount", "type": "uint256", "index": 1, "typeShort": "uint", "bits": "256", "displayName": "&thinsp;<span class=\"punctuation\">_</span>&thinsp;goal Amount", "template": "elements_input_uint", "value": "10000000000" } ], "payable": false, "stateMutability": "nonpayable", "type": "constructor", "signature": "constructor" } ]).at('0x164a9e6A2Ab7F0AaFfF1139490F837Fb9b2Bc7c7')
```

### Contract 생성
 - _duration : 테스트가 목적이므로 30분으로 설정 (미스티 월렛의 경우 30분이 1800)
 - _goalAmount : 목표액은 10ETH로 한다(wei) ( 미스티 월렛의 경우 10^19)
 - gas limit : 5000000
 
 ```
 투자자는 Contract에 이더를 보내는 형태의 거래를 만드는 방법으로 투자를 할 수 있다.
 Contract는 모금활동의 마감일과 목표액을 설정하고, 마감일 시점에서 목표액이 달성되면
 계약의 소유자에게 모금된 이더를 송금한다. 목표액이 달성되지 못했다면 투자자자에게
 이더를 돌려주게 된다.
 
(eth.accounts[0]) // 계약 소유자
(eth.accounts[1]) // 투자자 A
(eth.accounts[2]) // 투자자 B
 ```
 
 
### Scriipt

```
일단 채굴부터 시작하자.
> miner.start()
null

비밀번호 풀고..
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x072bbcdeafff45265e6d6e05225073c4c14e7e73
Passphrase:
true

> personal.unlockAccount(eth.accounts[1])
Unlock account 0x9d6c0d9814b733ef7f0605d374061e7246402bcd
Passphrase:
true

> personal.unlockAccount(eth.accounts[2])
Unlock account 0x92e735452e40569f21299138965de184b67bd401
Passphrase:
true

모금을 하려면 계정에 돈이 있어야 하므로..
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(10, "ether")})
"0x1f1fefe29ad412a3ad4ee296647a8d5c2cc5b8393d8e8cd32ac35a0254981453"

> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[2], value: web3.toWei(10, "ether")})
"0x9afc9c810528ddf90b49586f2fae407d12f2ccc5a4e677342cff2429644648bc"

투자자 A가 7ETH, 투자자 B가 3ETH를 fund 함수를 통해 투자함. 
sendTransaction으로 value값을 설정하면 이더를 송금하는 형태로 호출할 수 있음
> cf.fund.sendTransaction({from:eth.accounts[1], gas:5000000, value:web3.toWei(7,"ether")})
"0x25ff90f4cf79900d0a3bb7b00e87ff203a90efa36c46d1886ffd5c2aeb3742a9"

> cf.fund.sendTransaction({from:eth.accounts[2], gas:5000000, value:web3.toWei(3,"ether")})
"0x01ebeebc5ce8c614d90c2f5f964c248b452c6d5c663665c2097e6339c6301de2"

투자자 A의 투자액 확인
> web3.fromWei(cf.investors(0)[1], "ether")
7

투자자 B의 투자액 확인
> web3.fromWei(cf.investors(1)[1], "ether")
3

총 추자액 확인, totalAmount가 10ETH이므로 목표액이 달성했다.
> web3.fromWei(cf.totalAmount(), "ether")
10

계약 잔액 확인, 계약이 갖고 있는 이더의 액수 역시 같음을 알 수 있다.
> web3.fromWei(eth.getBalance(cf.address),"ether")
10

소유자의 잔액 확인 (이전에 소유자의 잔액을 기록해두어야 함)
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
7355.005293794

목표액이 달성했다면 소유자에게 모금된 이더가 송금된다. 투자자에게는 뭘 주지도 않음..;ㅣ
> cf.checkGoalReached.sendTransaction({from:eth.accounts[0], gas:5000000});
"0xbc39414a066da3609aed5f633bc12ba14e7f1d9f5a8048e2b699773d11d891cc"

상태확인
> cf.status()
"Campaign Succeeded"

> cf.ended()
true

잔액확인
> web3.fromWei(eth.getBalance(cf.address), "ether");
0

```

## 9-2 Managing names and addresses Contract

```c
pragma solidity ^0.4.11;
contract NameRegistry {

	// 컨트랙트를 나타낼 구조체
	struct Contract {
		address owner;
		address addr;
		bytes32 description;
	}


	// 등록된 레코드 수
	uint public numContracts;

	// 컨트랙트를 저장할 매핑
	mapping (bytes32  => Contract) public contracts;
    
	/// 생성자
	constructor() public{
		numContracts = 0;
	}

	/// 컨트랙트 등록
	function register(bytes32 _name) public returns (bool){
		// 아직 사용되지 않은 이름이면 신규 등록
		if (contracts[_name].owner == 0) {
			Contract storage con = contracts[_name];
			con.owner = msg.sender;
			numContracts++;
			return true;
		} else {
			return false;
		}
	}

	/// 컨트랙트 삭제
	function unregister(bytes32 _name) public returns (bool) {
		if (contracts[_name].owner == msg.sender) {
			contracts[_name].owner = 0;
 			numContracts--;
 			return true;
		} else {
			return false;
		}
	}
	
	/// 컨트랙트 소유자 변경
	function changeOwner(bytes32 _name, address _newOwner) public onlyOwner(_name) {
		contracts[_name].owner = _newOwner;
	}
	
	/// 컨트랙트 소유자 정보 확인
	function getOwner(bytes32 _name) constant public returns (address) {
		return contracts[_name].owner;
	}
    
	/// 컨트랙트 어드레스 변경
	function setAddr(bytes32 _name, address _addr) public onlyOwner(_name) {
		contracts[_name].addr = _addr;
    }
    
	/// 컨트랙트 어드레스 확인
	function getAddr(bytes32 _name) constant public returns (address) {
		return contracts[_name].addr;
	}
        
	/// 컨트랙트 설명 변경
	function setDescription(bytes32 _name, bytes32 _description) public onlyOwner(_name) {
		contracts[_name].description = _description;
	}

	/// 컨트랙트 설명 확인
	function getDescription(bytes32 _name) constant public returns (bytes32)  {
		return contracts[_name].description;
	}
    
	/// 함수를 호출 전 먼저 처리되는 modifier를 정의
	modifier onlyOwner(bytes32 _name) {
	    require(contracts[_name].owner == msg.sender);
		_;
	}
}
```

### Interface ABI

```
var nr= eth.contract([ { "constant": false, "inputs": [ { "name": "_name", "type": "bytes32" } ], "name": "unregister", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0x1a0919dc" }, { "constant": true, "inputs": [ { "name": "_name", "type": "bytes32" } ], "name": "getAddr", "outputs": [ { "name": "", "type": "address", "value": "0x0000000000000000000000000000000000000000" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x4ccee9b6" }, { "constant": true, "inputs": [], "name": "numContracts", "outputs": [ { "name": "", "type": "uint256", "value": "0" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x524d81d3" }, { "constant": false, "inputs": [ { "name": "_name", "type": "bytes32" }, { "name": "_description", "type": "bytes32" } ], "name": "setDescription", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0x980e1c5e" }, { "constant": true, "inputs": [ { "name": "_name", "type": "bytes32" } ], "name": "getDescription", "outputs": [ { "name": "", "type": "bytes32", "value": "0x0000000000000000000000000000000000000000000000000000000000000000" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0xb820a0ed" }, { "constant": false, "inputs": [ { "name": "_name", "type": "bytes32" }, { "name": "_newOwner", "type": "address" } ], "name": "changeOwner", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0xd3a5b107" }, { "constant": false, "inputs": [ { "name": "_name", "type": "bytes32" }, { "name": "_addr", "type": "address" } ], "name": "setAddr", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0xd5fa2b00" }, { "constant": true, "inputs": [ { "name": "_name", "type": "bytes32" } ], "name": "getOwner", "outputs": [ { "name": "", "type": "address", "value": "0x0000000000000000000000000000000000000000" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0xdeb931a2" }, { "constant": false, "inputs": [ { "name": "_name", "type": "bytes32" } ], "name": "register", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0xe1fa8e84" }, { "constant": true, "inputs": [ { "name": "", "type": "bytes32" } ], "name": "contracts", "outputs": [ { "name": "owner", "type": "address", "value": "0x0000000000000000000000000000000000000000" }, { "name": "addr", "type": "address", "value": "0x0000000000000000000000000000000000000000" }, { "name": "description", "type": "bytes32", "value": "0x0000000000000000000000000000000000000000000000000000000000000000" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0xec56a373" }, { "inputs": [], "payable": false, "stateMutability": "nonpayable", "type": "constructor", "signature": "constructor" } ]).at('0xcA37b661B4c838Eeefdce033aB459aa0951c9B82')
```

### Contract 생성

```
Name Registry는 계약의 이름과 어드레스를 관리하는 것이 목적인 계약이다. 
예를들어 하나 이상의 계약을 생성한다면 무작위 문자의 나열로 된 어드레스로 관리하는 것보다 
기억하기 쉬운 별명을 붙여두는 쪽이 직관적이고 관리도 쉬울것이다.

자연스러운 관리는 물론이고, 누군가에게 생성한 계약을 알려줘야 하는 경우에도 어드레스 대신 별명을 알려줌.
Name Registry에서 이 별명이 가리키는 계약을 조회하도록 하면 편리함.
또, 계약이 중간에 바뀌어도 별명이 접근접 역할을 해주므로 교체가 편리함.
```

```
각 계정의 역할을 다음과 같이 가정하고 계약을 생성해본다.

Name Registry가 아닌 계약 con1을 생성하고, 이 계약을 어드레스와 함께 Name Registry에 등록해둔다.
그리고 나서 con1의 이용자가 con1의 함수를 호출하기 위해 Name Registry에 어드레스를 조회하도록 한다.
만약 con1에 어떤 문제가 생겨도 새로운 con1을 만든 뒤, Name Registry 상의 어드레스를 새로운 어드레스로
업데이트 하는 방법으로 이용자들이 변화를 눈치채지 못하고 con1의 함수를 그대로 호출할 수 있다.
```


### Script

```
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x072bbcdeafff45265e6d6e05225073c4c14e7e73
Passphrase:
true

register 함수를 호출하고, name이 'con1'인 계약을 등록함.
> nr.register.sendTransaction("con1", {from:eth.accounts[0], gas:5000000})
"0x8c8b324eec5d69b777fd667ee0ece4909fa8b184565c538905859d67e4a1ad11"

numContracts의 값이 1이며, 'con1'의 소유자가 eth.account[0]이라는 것도 확인 가능
> nr.numContracts()
1

eth.account[0] : 0x072bbCDEaFff45265E6d6e05225073c4C14e7e73
> nr.getOwner("con1")
"0x072bbcdeafff45265e6d6e05225073c4c14e7e73"

con1에 해당하는 계약의 어드레스 설정. 어드레스는 무엇을 사용하든 상관없음. 설정이 끝나면 getAddr 함수로 등록여부 확인
> nr.setAddr.sendTransaction("con1", "0x072bbCDEaFff45265E6d6e05225073c4C14e7e73", {from:eth.accounts[0], gas:5000000})
"0xfe7e60a9d197e4edd512e699f10becce2ed2a6702bd869d1e0691b5cada3b1b1"

> nr.getAddr("con1")
"0x072bbcdeafff45265e6d6e05225073c4c14e7e73"

계약에 대한 추가설명을 위해 setDescription 함수를 호출, 문자열 추가, getDescription 함수를 사용하여 확인
> nr.setDescription.sendTransaction("con1", "this is for con1", {from:eth.accounts[0], gas:5000000})
"0x419df485f99b6fe6f679224ab964d99abb870918bf637fa699ff78fe7a6ee66c"

> nr.getDescription("con1")
"0x7468697320697320666f7220636f6e3100000000000000000000000000000000"

> web3.toUtf8(nr.getDescription("con1"))
"this is for con1"

changerOwner를 호출하여 con1의 소유자를 eth.accounts[1]로 변경
> nr.changeOwner.sendTransaction("con1", eth.accounts[1], {from:eth.accounts[0], gas:5000000})
"0x015d84e36191aaaa0665ef0c373e1270322739769163fdf42ff28ebad5610fe9"

eth.account[1] : 0x9d6C0d9814b733EF7f0605D374061e7246402Bcd 확인
> nr.getOwner("con1")
"0x9d6c0d9814b733ef7f0605d374061e7246402bcd"

> personal.unlockAccount(eth.accounts[1])
Unlock account 0x9d6c0d9814b733ef7f0605d374061e7246402bcd
Passphrase:
true

마지막으로, unregister 함수로 'con1'을 Name Registry에서 제거한다.
> nr.unregister.sendTransaction("con1", {from:eth.accounts[1], gas:5000000})
"0x2cbaea6a5be66a96cae2cb78b6f381e6b3c243c2bc40ca4da9cb7a205b13bc52"

numContracts가 0이 되었음을 
> nr.numContracts()
0
```

## 9-3 Contract to control IoT switch

```c
pragma solidity ^0.4.11;
contract SmartSwitch {
	// 스위치가 사용할 구조체
	struct Switch {
		address addr;	// 이용자 어드레스
		uint	endTime;	// 이용 종료 시각 (UnixTime)
		bool 	status;	// true이면 이용 가능
	}
	
	address public owner;	// 서비스 소유자 어드레스
	address public iot;	// IoT 장치의 어드레스
		
	mapping (uint => Switch) public switches;	// Switch 구조체를 담을 매핑
	uint public numPaid;			// 결제 횟수
	
	/// 서비스 소유자 권한 체크
	modifier onlyOwner() {
		require(msg.sender == owner);
		_;
	}
	
	/// IoT 장치 권한 체크
	modifier onlyIoT() {
		require(msg.sender == iot);
		_;
	}
	
	/// 생성자
	/// IoT 장치의 어드레스를 인자로 받음
    constructor(address _iot) public{
		owner = msg.sender;
		iot = _iot;
		numPaid = 0;
	}

	/// 이더를 지불할 때 호출되는 함수
	function payToSwitch() public payable {
		// 1 ETH가 아니면 처리 종료
		require(msg.value == 1000000000000000000);
		
		// Switch 생성
		Switch storage s = switches[numPaid++];
		s.addr = msg.sender;
		s.endTime = now + 300;
		s.status = true;
	}
	
	/// 스테이터스를 변경하는 함수
	/// 이용 종료 시각에 호출됨
	/// 임자는 switches의 키 값
	function updateStatus(uint _index) public onlyIoT {
		// 인덱스 값에 해당하는 Switch 구조체가 없으면 종료
		require(switches[_index].addr != 0);
		
		// 이용 종료 시각이 되지 않았으면 종료
		require(now > switches[_index].endTime);
		
		// 스테이터스 변경
		switches[_index].status = false;
	}

	/// 지불된 이더를 인출하는 함수
	function withdrawFunds() public onlyOwner {
		if (!owner.send(address(this).balance)) 
			revert();
	}
	
	/// 컨트랙트를 소멸시키는 함수
	function kill() public onlyOwner {
		selfdestruct(owner);
	}
}
```

### Interface ABI

```
var ss= eth.contract([ { "constant": true, "inputs": [], "name": "iot", "outputs": [ { "name": "", "type": "address", "value": "0x9d6C0d9814b733EF7f0605D374061e7246402Bcd" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x113a0cfe" }, { "constant": false, "inputs": [], "name": "withdrawFunds", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0x24600fc3" }, { "constant": true, "inputs": [], "name": "numPaid", "outputs": [ { "name": "", "type": "uint256", "value": "0" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x2bdcc167" }, { "constant": false, "inputs": [], "name": "kill", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0x41c0e1b5" }, { "constant": true, "inputs": [ { "name": "", "type": "uint256" } ], "name": "switches", "outputs": [ { "name": "addr", "type": "address", "value": "0x0000000000000000000000000000000000000000" }, { "name": "endTime", "type": "uint256", "value": "0" }, { "name": "status", "type": "bool", "value": false } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x881ec10b" }, { "constant": false, "inputs": [], "name": "payToSwitch", "outputs": [], "payable": true, "stateMutability": "payable", "type": "function", "signature": "0x8b744891" }, { "constant": true, "inputs": [], "name": "owner", "outputs": [ { "name": "", "type": "address", "value": "0x072bbCDEaFff45265E6d6e05225073c4C14e7e73" } ], "payable": false, "stateMutability": "view", "type": "function", "signature": "0x8da5cb5b" }, { "constant": false, "inputs": [ { "name": "_index", "type": "uint256" } ], "name": "updateStatus", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function", "signature": "0xd61e7201" }, { "inputs": [ { "name": "_iot", "type": "address", "index": 0, "typeShort": "address", "bits": "", "displayName": "&thinsp;<span class=\"punctuation\">_</span>&thinsp;iot", "template": "elements_input_address", "value": "0x9d6C0d9814b733EF7f0605D374061e7246402Bcd" } ], "payable": false, "stateMutability": "nonpayable", "type": "constructor", "signature": "constructor" } ]).at('0x38546a960270746a1cD4f4E0DDB76EFD7712fde2')
```

### Contract 생성

```
최근 널리 이용되는 카 셰어링에 이를 적용해본다. 
이용자는 자동차를 이용하기 위해 이용시간에 따른 금액을 지불해야 한다. 
이때 스마트 계약에 송금하면 자동차가 송금 상황을 확인하고 잠긴 차문을 열어주는 것을 생각해볼 수 있다.
```

```
각 계정의 역할을 다음과 같이 가정하고 계약을 생성해본다.
eth.accounts[0] : 서비스 소유자
eth.accounts[1] : IoT 장치
eth.accounts[2] : 이용자

1. 먼저 서비스 소유자가 SmartSwitch를 생성한다.
2. IoT 이용자는 payToWsitch 함수로 이더를 송금한다.
3. IoT 장치가 송금을 확인한 다음 서비스를 시작한다.
4. 이용시간이 종료되면 IoT 장치가 다시 UpdateStatus 함수를 이용하여 스테이터스를 변경하고 서비스 이용을 마친다.
5. 서비스 소유자는 withdrawFunds 함수르 SmartSwitch에 송금된 이더를 수납한다.
```


### Script

```
어드레스 확인
> ss.owner()
"0x072bbcdeafff45265e6d6e05225073c4c14e7e73"

> ss.iot()
"0x9d6c0d9814b733ef7f0605d374061e7246402bcd"

switch 값 확인
> ss.switches(0)
["0x0000000000000000000000000000000000000000", 0, false]

> personal.unlockAccount(eth.accounts[2])
Unlock account 0x92e735452e40569f21299138965de184b67bd401
Passphrase:
true

이용 등록(payToSwitch)
이용자의 어드레스에서 1eth를 송금하도록 payToSwitch 함수를 호출하고 이용등록을 마침
> ss.payToSwitch.sendTransaction({from:eth.accounts[2], gas:5000000, value:web3.toWei(1,"ether")})
"0xfe65b59f433167fe22b602973816fb414821afdbad2f63d8c9c4e3ebd74e06c6"

switches 매핑의 키 0에 등록된 정보를 확인할 수 있음. 
switches에 등록된 값은 구조체 true임을 확인하고 IoT장치를 사용할 수 있게됨
> ss.switches(0)
["0x92e735452e40569f21299138965de184b67bd401", 1557828046, true]

결제 횟수가 1임을 확인
> ss.numPaid()
1

계약 잔액 확인 1ETH
> web3.fromWei(eth.getBalance(ss.address),"ether")
1

이용 종료 시 updateStatus 호출
이용 종료 시각이 되면 IoT장치가 updateStatus함수를 호출한다. (수동임..;)
//5분 후에 이용, IoT 장치도 gas 를 보낼 돈은 가지고 있어야함(주의)
> ss.updateStatus.sendTransaction(0, {from:eth.accounts[1], gas:5000000})
"0x79bac33844196e33593f531b992f37e2997e8762c44301c3a99fd113c66e5b7c"

스테이터스 변경 결과 확인 true -> false
> ss.switches(0)
["0x92e735452e40569f21299138965de184b67bd401", 1557828046, false]

이더 수납 (서비스 소유자는 계약이 갖고 있던 이더를 가저간다)
> ss.withdrawFunds.sendTransaction({from:eth.accounts[0], gas:5000000})
"0xb5e1e449739da3265d82ad7d2b1646c41d68519b08280f81e5c1f97df9a733a8"

계약 잔액 확인 : 0
> web3.fromWei(eth.getBalance(ss.address),"ether")
0
```
