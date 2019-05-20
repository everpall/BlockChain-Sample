### 11.5 오버플로
 > 11.2 절에서 다뤘던 MarketPlace 예제를 추가 (재고를 추가하는 함수를 추가 구현)

> MarketPlaceOverflow.sol
```c
pragma solidity ^0.4.11;
contract MarketPlaceOverflow {
	address public owner;
	uint8 public stockQuantity;	// 재고 수
	
	modifier onlyOwner() {
		require(msg.sender == owner);
		_;
	}
	
	/// 추가 재고 수를 출력하는 이벤트
	event AddStock(uint _addedQuantity);
			
	/// 생성자
	constructor() public{
		owner = msg.sender;
		stockQuantity = 100;
	}
	
	/// 재고를 추가하는 처리
	function addStock(uint8 _addedQuantity) public onlyOwner {
		emit AddStock(_addedQuantity);
		stockQuantity += _addedQuantity;
	}
}
```

인수로 받은 _addQuantity 값을 현재 재고인 stockQuantity에 더하는 형태로 현재 재고를 수정하는 내용이다.

```
초기 재고 100
> mpof.stockQuantity()
100

재고 156개 추가
> mpof.addStock.sendTransaction(156,{from:eth.accounts[0], gas:5000000})
"0xb7b6a47dd3c21eeba6ac9577e90626867fc6a7c086adf6d773a984f180c14c3b"

어떻게 된것인가?
> mpof.stockQuantity()
0

100으로 되돌린뒤 다시 시작
> mpof.addStock.sendTransaction(100,{from:eth.accounts[0], gas:5000000})
"0x362edd631db52bbd3da8684a074516228d02b6ecc032ef03bd5b53596f6df320"

> mpof.stockQuantity()
100

> mpof.addStock.sendTransaction(157,{from:eth.accounts[0], gas:5000000})
"0x195cf371fecb9200bbaaf0407737c3b319d365c44232b6125f98294b1ff265e2"

> mpof.stockQuantity()
1

```

### 1차 수정하기
>MarketPlaceOverflowMod.sol
```c
/// 재고를 추가하는 처리
	function addStock(uint8 _addedQuantity) public onlyOwner {
		// 오버플로우 확인
		require(stockQuantity + _addedQuantity > stockQuantity);
		
		emit AddStock(_addedQuantity);
		stockQuantity += _addedQuantity;
	}
```

수정 사항 : 추가된 재고가 원래 재고보다 많은지 확인하는 방식으로 오버플로를 방지하는 방법

```
> mpofmod.addStock.sendTransaction(157,{from:eth.accounts[0], gas:5000000})
"0x052b54515659ba1272c57f1eeac6dcf0814e697993825c5303c7e85b4072a417"

이번에는 1이 아니라 100으로 그대로 남아있는 것을 알 수 있다.
> mpofmod.stockQuantity()
100

gasUsed를 확인하면 예외가 발생했음을 짐작할 수 있다. (이 부분은 이해가 안감.. gasUsed로 어떻게 판단을?)
> eth.getTransactionReceipt('0x052b54515659ba1272c57f1eeac6dcf0814e697993825c5303c7e85b4072a417').gasUsed
5000000

```

이는 다음 조건식이 false가 되었기 때문이다.

stockQuantity + _addedQuantity > stockQuantity

설명하자면 다음과 같다.
  * 좌변 : 100 + 157 = 257(오버플로 발생) = 1
  * 우변 : 100

이번에는 재고를 257개로 늘려본다.

```
> mpofmod.addStock.sendTransaction(257,{from:eth.accounts[0], gas:5000000})
"0x1eb68e4d7c2031685e75640a2591fd6a988f2d3b41a8b5b0233e43ba31039aba"

> mpofmod.stockQuantity()
101

> eth.getTransaction('0x1eb68e4d7c2031685e75640a2591fd6a988f2d3b41a8b5b0233e43ba31039aba').input
"0xad9de1ee0000000000000000000000000000000000000000000000000000000000000101"
```

uint8로는 255까지 밖에 나타내지 못하므로 100+257 = 357 이니까 예외가 발생해야 하는데, 결과는 재고가 1 증가한 것으로 나왔다. 
input을 확인해보면 마지막 세 줄에 101 이 나왔고, 이를 10진수로 바꾸면 257이다.
addStock 함수는 인자가 uint8로 선언되어 있어도 이더리움에서는 이 길이를 초과하는 값을 지정할 수 있다.
그렇기 때문에 uint8의 표현 범위를 초과하는 값(257)을 인자로 지정하여 거래를 생성할 수는 있지만,
실제 함수에 넘겨지는 인자는 01 이므로 예외가 발생하지 않는다.

따라서 조건식이 참이되는 것이다. (101 > 100)

### 2차 코드 수정

```c
/// 재고를 추가하는 처리
	function addStock(uint8 _addedQuantity) public onlyOwner {
		// 재고 수 확인
		require(_addedQuantity < 256); // 추가된 코드
		
		// 오버플로우 확인
		require(stockQuantity + _addedQuantity > stockQuantity);
		
		emit AddStock(_addedQuantity);
		stockQuantity += _addedQuantity;
	}
```

```
> mpofmod2.addStock.sendTransaction(257,{from:eth.accounts[0], gas:5000000})
"0xa45845ba9e2ba86010535736c83a1438e4be026312ef27bf497299706ab215b7"

> mpofmod2.stockQuantity()
101
```

이번에도 예외가 발생하지 않았다. 아까와 같은 이유로 addStock(uint8 _addedQuantity) 의 인자가 넘올때 자릿수가 누락됬기 때문

### 3차 코드 수정
> uint8 -> uint
```c
/// 재고를 추가하는 처리
	function addStock(uint _addedQuantity) public onlyOwner {  // 수정된 부분
		// 재고 수 확인
		require(_addedQuantity < 256);
		
		// 오버플로우 확인
		require(stockQuantity + uint8(_addedQuantity) > stockQuantity);
		
		emit AddStock(_addedQuantity);
		stockQuantity += uint8(_addedQuantity);
	}

```

```
> mpofmod3.addStock.sendTransaction(257,{from:eth.accounts[0], gas:5000000})
"0x490838b1102d7b8edf227fc743caf20fd5b4bb3e2f03a4b9a205b35f4f86285d"

> mpofmod3.stockQuantity()
100
```

이번에는 재고가 바뀌지 않았다. 
지금까지 확인한 내용대로라면 stockQuantity 인자로 uint8 대신 256비트 길이인 uint으로 선언하는 것이 나아 보인다.
그러나 uint의 표현 범위를 넘는 값이 인자로 주어지면 결국 마찬가지일 것이다.

### 4차 코드 수정
>편의상 생성자에서 stockQuantity 의 초깃값을 0으로 설정
```c
/// 생성자
	constructor() public{
		owner = msg.sender;
		stockQuantity = 0;
	}
  
/// 재고를 추가하는 처리
	function addStock(uint _addedQuantity) public onlyOwner {
		// 재고 수 확인
		require(stockQuantity + _addedQuantity > stockQuantity);
		
		emit AddStock(_addedQuantity);
		stockQuantity += _addedQuantity;
	}
```

```
2^256개 추가
> mpofmod4.addStock.sendTransaction("0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",{from:eth.accounts[0],gas:5000000})
"0x7d97a09c139b05bb95739c761208283decb51b0e17d1a54cf9396800c0ede4a4"

상태 확인
> mpofmod4.stockQuantity().toFixed()
"115792089237316195423570985008687907853269984665640564039457584007913129639935"

1개 추가
> mpofmod3.addStock.sendTransaction(1,{from:eth.accounts[0], gas:5000000})
"0x567b4fce42151c9551c7565cf9fade50ef7a5bc67f8aaee596edbb788d5682d5"

재고가 추가되지 않았다.
> mpofmod4.stockQuantity().toFixed()
"115792089237316195423570985008687907853269984665640564039457584007913129639935"
```

uint 는 표현 범위를 초과하는 값을 설정하려고 하면 오류가 발생하여 거래가 발행되지 않으므로, 따로 확인할 필요는 없다고 한다.
