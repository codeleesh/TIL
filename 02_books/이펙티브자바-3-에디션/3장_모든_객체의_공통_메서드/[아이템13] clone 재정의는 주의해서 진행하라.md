# [아이템13] clone 재정의는 주의해서 진행하라

이 내용은 `이펙티브 자바 Effective Java 3/E` 를 읽으면서 정리한 내용을 포함하고 있습니다.

`clone` 메서드의 목적은 객체의 복제본(원본과 상태가 같은 별개의 객체)을 만드는 것입니다. 



## clone 구현

clone 메서드를 정상적으로 구현하기 위해서는 `Cloneable` 인터페이스를 구현해야 합니다.

```java
@HotSpotIntrinsicCandidate
protected native Object clone() throws CloneNotSupportedException;
```



### clone 메서드를 구현하지 않아도 되는가?

clone은 Object 클래스에 정의되어 있습니다. 객체가 clone 메서드를 상속해도 clone 메서드의 접근제어자는 protected 이므로 메서드를 호출할 수 없습니다.



### clone 메서드를 재정의해야 하는가?

clone을 사용하기 위해서 `Cloneable` 인터페이스를 구현해야 합니다. 그리고 접근제어자는 proctected 에서 public 으로 변경해야 합니다. 또한 CloneNotSupportedException를 처리해야 합니다. Object.clone() 메서드는 얕은 복사를 수행하기에 앞서 클래스가 Cloneable 인터페이스를 구현했는지 검사하고, 구현하지 않았다면 CloneNotSupportedException을 던지기 때문입니다.

아래는 `Cloneable` 인터페이스를 구현하지 않고 `clone` 메서드만 구현해서 호출하는 테스트를 해보았습니다.

```java
@Override
public Object clone() throws CloneNotSupportedException {
	return super.clone();
}
```

`Cloneable` 인터페이스를 구현하지 않고 `clone` 메서드만 구현해서 호출하면 `CloneNotSupportedException` 으로 반환됩니다.

```java
@DisplayName("Cloneable 인터페이스를 구현하지 않고 clone 메서드를 호출하면 CloneNotSupportedException 된다.")
@Test
void cloneTest() throws CloneNotSupportedException {

	PhoneNumber phoneNumber = PhoneNumber.of(707, 867, 5309);

	assertThatThrownBy(() -> phoneNumber.clone()).isInstanceOf(CloneNotSupportedException.class);
}
```

- 실제 환경에서 clone을 사용한다면 해당 객체가 Cloneable을 구현해야 사용이 가능하기 때문에 무조건 성공할 것이며, `CloneNotSupportedException` 이 나타날 경우는 거의 없다고 생각이 됩니다.



`Cloneable` 인터페이스를 구현하고 `clone` 메서드만 구현해서 호출하면 원하는 객체로 반환이 됩니다.

Object의 clone 메서드는 Object를 반환하지만 우리가 사용할 객체의 clone 메서드는 해당 객체를 반환하게 해야 합니다. 자바가 공변 변환 타이핑을 지원하기 때문에 가능한 방식입니다.

`공변 변환 타입`이란? 

- 부모 클래스의 메소드를 오버라이딩하는 경우, **부모 클래스의 반환 타입은 자식 클래스의 타입으로 변경이 가능**합니다.
- Object의 메소드인 clone을 오버라이딩하였기 때문에, PhoneNumber로 반환 타입이 가능합니다.



## clone의 문제점

가변 객체를 참조하는 순간 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해치게 됩니다. 프로그램이 이상하게 동작할 수 있게 되며, 원인을 찾기가 굉장히 어려워집니다.

또한 final 필드와도 충돌할 수 있으며, 불필요한 검사 예외를 던지고, 불필요한 형변환도 합니다.



## clone 사용해야 한다면

- `Cloneable` 을 구현하는 모든 클래스는 clone을 재정의해야 합니다.

- 접근제한자는 protected에서 public으로 변경해야 합니다.

- 반환 타입은  object에서 클래스 자신으로 변경해야 합니다.

- clone 메서드는 super.clone 호출해야 합니다.



이러한 방법을 모두 따라서 꼭 clone을 사용해야 할까요? 단, 배열인 경우 clone 메서드 방식이 가장 깔끔합니다.



## clone 대체

clone의 문제점을 해결할 수 있고, 불필요한 인터페이스를 구현하지 않아도 되는 방법이 있습니다.

바로 복사 생성자와 복사 팩터리 메소드를 사용하다면 더 나은 객체 복사 방식을 사용할 수 있습니다.



## 정리

- `clone` 메서드를 사용하기 위해서는 `Cloneable` 인터페이스를 구현해야 하며, 접근제한자도 변경해야 하고 변환 타입도 변경해야 합니다.
- `clone` 의 문제점을 생각한다면 복사 생성자와 복사 팩터리 메서드를 사용해서 객체를 복사할 수 있습니다.