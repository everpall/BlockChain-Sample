### 11.3 Timestamp Dependence
> 블록의 타임스탬프 값에 의존하는 처리를 공격하는 취약점도 자주 발생

> block.timestamp는 블록이 생성된 시점의 Unixtime 값을 갖고 있음

### 로또 당첨 시스템
> Lottery.sol

```c
pragma solidity ^0.4.11;
contract Lottery {
	// 응모자를 관리하는 매핑
	mapping (uint => address) public applicants;

	// 응모자 수
	uint public numApplicants;

	// 당첨자 정보
	address public winnerAddress;
	uint public winnerInd;
	
	// 소유자
	address public owner;

	// 타임스탬프
	uint public timestamp;
	
	/// 소유자 여부를 확인하는 modifier
	modifier onlyOwner() {
		require(msg.sender == owner);
		_;
	}
	
	/// 생성자
	constructor() public{
		numApplicants = 0;
		owner = msg.sender;
	}

	/// 추첨 응모 처리 함수
	function enter() public {
		// 응모자가 3명 미만인지 확인
		require(numApplicants < 3);
		
		// 이미 응모한 사람이 아닌지 확인
		for(uint i = 0; i < numApplicants; i++) {
			require(applicants[i] != msg.sender);
		}
		
		// 응모 접수 처리
		applicants[numApplicants++] = msg.sender;
	}
	
	/// 추첨
	function hold() public onlyOwner {
		// 응모자가 3명 이상인지 확인
		require(numApplicants == 3);
		
		// 타임스탬프 값 설정
		timestamp = block.timestamp;
		
		// 추첨
		winnerInd = timestamp % 3;
		winnerAddress = applicants[winnerInd];
	}

}
```

* 계정 정보
  * eth.accounts[0] : 계약 생성자이자 추첨 주최자
  * eth.accounts[1] ~ eth.account[3] : 응모자
  
### Script

```
추첨 응모 : 3개의 어드레스에서 응모가 이루어졌음을 확인
> lot.enter.sendTransaction({from:eth.accounts[1], gas:5000000})
"0x0cd2ef2129637d16d3d967e32f24a7ed335e8ed3ef002505985a31cb56edc50a"
> lot.enter.sendTransaction({from:eth.accounts[2], gas:5000000})
"0xe158b8459fbd8acf00b339b949f1a26f8285d11977bdd9db1b7a4a47814ad9dd"
> lot.enter.sendTransaction({from:eth.accounts[3], gas:5000000})
"0x3ba09384871c517b10ecaa95760cb315e12de52946895bb5b1dddf37a44a2af0"

> lot.numApplicants()
3

> lot.applicants(0)
"0x9d6c0d9814b733ef7f0605d374061e7246402bcd"
> lot.applicants(1)
"0x92e735452e40569f21299138965de184b67bd401"
> lot.applicants(2)
"0x49f1bc2655aff939412b562262bb4a65baad7c4b"

hold함수를 이용하여 당첨자 추첨
> lot.hold.sendTransaction({from:eth.accounts[0], gas:5000000})
"0x192929a210917047129f5e1833ca36541610755861a6dd6b1eeefe603fde3c09"

winnerInd가 2이므로 2번 응모자가 당첨되었음을 알 수 있다.
> lot.winnerInd()
2

2번 응모자 당첨 확인
> lot.winnerAddress()
"0x49f1bc2655aff939412b562262bb4a65baad7c4b"

hold를 실행했던 거래의 해시를 이용하여 이 거래가 저장되 블록의 blockNumber확인
확인된 blockNumber에 해당하는 블록이 생성된 timestamp확인
> eth.getBlock(eth.getTransactionReceipt('0x192929a210917047129f5e1833ca36541610755861a6dd6b1eeefe603fde3c09').blockNumber).timestamp
1558335347

Lotter 계약의 timestamp 확인
> lot.timestamp()
1558335347
```

### 취약점 원리
> 거래가 발행된 시점이 아니라, 채굴자가 블록을 생성한 시점의 Unixtime 값을 설정하므로 이 값은 채굴자에 의존한다고 할 수 있음

> 채굴자는 timestamp 값을 어느정도 통제할 수 있기 때문에 악의적인 의도를 갖고 이 값을 결정할 수 있음.

> 예를들면 winnerInd 는 block.timestamp를 3으로 나는 나머지 이므로 채굴자가 이에 적절한 timestamp를 결정할 수 있다.

따라서 timestamp 만으로 모든 것이 결정되게 해서는 안되며, 결과는 무작위 성을 유지하기 위해 다른 요소도 고려해야 한다.
