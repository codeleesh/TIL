# [아이템12] toString을 항상 재정의하라 

이 내용은 `이펙티브 자바 Effective Java 3/E` 를 읽으면서 정리한 내용을 포함하고 있습니다.

객체의 기본 toString 메서드는 적합한 문자열을 반환하는 경우는 거의 없습니다.



## toString 재정의 필요한 이유

아래 PhoneNumber 객체는 toString 을 재정의하지 않았습니다. 즉 기본 toString을 호출하도록 되어 있습니다.

```java
PhoneNumber phoneNumber = PhoneNumber.of(707, 867, 5309);
System.out.println("phoneNumber : " + phoneNumber);
```

결과는 다음과 같습니다. 

```
phoneNumber : item11.domain.PhoneNumber@b501c
```

- phoneNumber의 어떠한 값이 있는지 알아보기가 힘듭니다.

에러가 발생했다고 가정하고, 시스템의 로그가 이렇게 나온다면 원인 파악이 가능할까요?

toString을 제대로 재정의하지 않았기 때문에 쓸모없는 메시지만 로그에 남게 됩니다.



다음은 toString을 재정의하였습니다.

```java
@Override
public String toString() {
	return "PhoneNumber{" +
		"phone1=" + phone1 +
		", phone2=" + phone2 +
		", phone3=" + phone3 +
	'}';
}
```

재정의한 결과는 다음과 같습니다.

```
phoneNumber : PhoneNumber{phone1=707, phone2=867, phone3=5309}
```

toString을 잘 구현한 클래스는 사용하기에 훨씬 좋고, 그 클래스를 사용한 시스템은 디버깅하기 쉽습니다.



## 출력 정보

우리가 마주하는 실제 개발환경에서 toString은 그 객체가 가진 주요 정보를 모두 반환하는게 좋습니다. 단 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 요약 정보를 담아야 합니다. 

단, 요약 정보의 경우 포맷이 정해지면 해당 객체를 사용하는 부분들도 의존하게 된다는 것을 명심해야 합니다.



## API 제공

toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 합니다.

위에서 예시로 사용한 PhoneNumber 객체는 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야 합니다. 제공하지 않는다면, 이 정보가 필요한 개발자는 toString의 반환값을 파싱해서 사용하게 됩니다.



## toString 재정의 방법

toString의 재정의는 equals과 hashCode와 동일하게 Lombok과 IDE를 통해서 할 수 있습니다.

- Lombok의 `@ToString` 어노테이션 사용
- Intellij에서 작성한 클래스에서 단축키(맥 OS : `Command + N`, 윈도우 : `alt + Insert`) 를 이용하여서 생성할 수 있습니다.



## 정리

- 객체의 기본 toString은 적합한 객체의 정보를 반환하지 않기 때문에 재정의를 해야합니다.
  - toString을 재정의한 클래스를 사용하는 시스템은 디버깅하기 쉽습니다.
- toString 재정의 방법은 `Lombok` 과 IDE에서 생성하는 toString을 통해서 재정의할 수 있습니다.