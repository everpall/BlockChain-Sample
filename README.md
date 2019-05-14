# BlockChain-Sample
Block chained for engineers

### Environment Setting

```
cd C:\Users\Sunghyun\AppData\Roaming\Ethereum Wallet\binaries\Geth\geth-windows-amd64-1.8.23-c9427004

geth --datadir home\eth_private_net init home\eth_private_net\genesis.json

geth --networkid "10" --nodiscover --datadir "home\eth_private_net" --rpc --rpcaddr "localhost" --rpcport "8545" --rpccorsdomain "*" --rpcapi "eth,net,web3,personal" --targetgaslimit "20000000" console 2>> home\eth_private_net\geth_err.log
```

### 9-2 Crowd fundding

```
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
