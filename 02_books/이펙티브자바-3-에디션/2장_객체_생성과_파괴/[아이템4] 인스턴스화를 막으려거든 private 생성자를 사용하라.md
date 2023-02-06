# [아이템4] 인스턴스화를 막으려거든 private 생성자를 사용하라

이 내용은 `이펙티브 자바 Effective Java 3/E` 를 읽으면서 정리한 내용을 포함하고 있습니다.



## 정적 메소드와 정적 필드

정적 메소드와 정적 필드만을 담은 클래스를 사용할때가 있습니다.

- `java.lang.Math` , `java.util.Arrays` 처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓았습니다.
- `java.util.Collections` 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메소드를 모아놓았습니다.
  - 자바 8부터는 정적 메소드를 인터페이스로 생성 가능합니다.
- final 클래스와 관련한 메소드들을 모아놓았습니다.



정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아닙니다. 하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어줍니다. 

즉, 매개변수를 받지 않는 public 생성자가 만들어지며, 사용자는 이 생성자가 자동 생성된 것인지 구분할 수 없습니다.



## 추상 클래스 인스턴스화

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없습니다. 아래 Car 추상 클래스와 Xm3  하위 클래스 예시를 보도록 하겠습니다.

```java
public abstract class Car {

    public static void drive(String message) {
      
        System.out.println(message);
    }
}

public class Xm3 extends Car {

    public Xm3() {
        super();
    }
}
```

- 하위 클래스인 `Xm3` 에서 인스턴스화를 할 수 있습니다.
- 사용자 입장에서는 상속해서 사용해도 된다고 생각할 수 있습니다.



### 해결책

하지만 이를 막는 방법은 간단합니다. 

컴파일러가 기본 생성자를 만드는 경우는 오직 명시된 생성자가 없을 때뿐이니 private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있습니다.

```java
public abstract class Car {

		private Car() {
		
				throw new AssertionError();
		}

   	public static void drive(String message) {
   	
        System.out.println(message);
    }
}
```

- 명시적 생성자가 private 이니 클래스 바깥에서는 접근할 수 없습니다.



## 정리

- 정적 메소드와 정적 필드만을 담은 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만듭니다.
- 추상 클래스로 만드는 것으로는 인스턴스화를 막기 위한 간단한 방법은 기본 생성자를 private 으로 생성하는 것입니다.