# [Unit Testing] 3장 단위 테스트 구조





## 단위 테스트를 구성하는 방법





### AAA 패턴 사용

AAA 패턴은 각 테스트를 준비, 실행, 검증이라는 세 부분으로 나눌 수 있습니다.

- Given-When-Then 패턴과 유사합니다.

이 패턴의 가장 큰 장점은 일관성이라는 부분입니다. 익숙해지면 모든 테스트를 쉽게 읽을 수 있고 이해할 수 있습니다.

- 준비 구절에서는 테스트 대상 시스템과 해당 의존성을 원하는 상태로 만듭니다.
- 실행 구절에서는 테스트 대상 시스템에서 메서드를 호출하고 준비된 의존성을 전달하며 출력 값을 캡처한다.
- 검증 구절에서는 결과를 검증합니다. 결과는 반환 값이나 테스트 대상 시스템과 협력자의 최종 상태, 협력자에 호출한 메서드 등으로 표시될 수 있습니다.



#### 테스트 작성 방법

- 보통의 테스트 코드 작성할 때
  - 테스트 작성 시 준비 구절부터 시작하는 것이 자연스럽습니다. 그다음 다른 두 구절을 작성합니다.
- 테스트 주도 개발(TDD) 실천할 때
  - 기능을 개발하기 전에 실패할 테스트를 만들 때는 아직 기능이 어떻게 동작할지 충분히 알지 못합니다.
  - 먼저 기대하는 동작으로 윤곽을 잡으면서 개발을 하는 것입니다.



### 여러 개의 준비, 실핼, 검증 구절 피하기

준비, 실행 또는 검증 구절이 여러 개 있는 테스트를 만날 수 있습니다. 이러한 테스트는 단위 테스트가 아닌 통합 테스트 입니다. 이러한 단위 테스트 구조는 피하는 것이 좋습니다. 

실행이 하나인 경우 테스트의 장점은 다음과 같습니다.

- 테스트가 단위 테스트 범주에 있게끔 보장하고, 간단하고, 빠르며, 이해하기 쉽습니다.

통합 테스트의 경우에는 실행을 여러 개 두는 것이 좋을 수 있습니다.



### 테스트 내 if 문 피하기

if 문이 있는 단위 테스트는 안티 패턴입니다. 단위 테스트든 통합 테스트든 테스트는 분기가 없는 간단한 일련의 단계여야 합니다.

if 문은 테스트가 한 번에 너무 많은 것을 검증한다는 표시입니다. 이러한 테스트는 반드시 여러 테스트로 나눠야 합니다.

if 문은 테스트를 읽고 이해하는 것을 더 어렵게 만들게 됩니다.



### 각 구절은 얼마나 커야 하는가

준비 구절에서 재사용에 도움이 되는 두 가지 패턴이 있습니다.

- 오브젝트 마더
- 테스트 데이터 빌더



실행 구절은 보통 코드 한 줄입니다. 실행 구절이 두 줄 이상인 경우 문제가 있을 수 있습니다.

두 줄로 실행된 구절입니다.

```java
@Test
void Purchase_succeeds_when_enough_inventory_2() {

	Store store = new Store();
	store.addInventory(Product.Shampoo, 10);
	Customer customer = new Customer();

	boolean success = customer.purchase(store, Product.Shampoo, 5);
	store.removeInventory(success, Product.Shampoo, 5);

	assertAll(
    () -> assertThat(success).isTrue(),
		() -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(5)
	);
}
```

- 구매를 마치려면 두 번째 메서드를 호출해야 하므로, 캡슐화가 깨지게 됩니다.
- 비즈니스 관점에서 구매가 정상적으로 이뤄지면 고객의 재품 획득과 매장 재고 감소라는 두 가지 결과가 만들어지게 됩니다.
  - 이러한 결과는 같이 만들어야 하고 이는 단일한 공개 메서드가 있어야 합니다.

한 줄로 실행된 구절입니다.

```java
@Test
void Purchase_succeeds_when_enough_inventory() {

  Store store = new Store();
	store.addInventory(Product.Shampoo, 10);
	Customer customer = new Customer();

	boolean success = customer.purchase(store, Product.Shampoo, 5);

	assertAll(
    () -> assertThat(success).isTrue(),
		() -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(5)
	);
}
```



이러한 모순을 불변 위반이라고 하며, 잠재적 모순으로부터 코드를 보호하는 행위를 캡슐화라고 합니다. 코드 캡슐화를 항상 지키는 것을 신경써야 합니다.
