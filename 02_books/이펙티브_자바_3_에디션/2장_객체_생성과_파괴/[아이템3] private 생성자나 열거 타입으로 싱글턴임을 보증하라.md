# [아이템3] private 생성자나 열거 타입으로 싱글턴임을 보증하라

이 내용은 `이펙티브 자바 Effective Java 3/E` 를 읽으면서 정리한 내용을 포함하고 있습니다.

내용을 알아보기 전, 싱글턴에 대해서 알고 넘어가도록 하겠습니다.



## 싱글턴

- 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말합니다.

- 싱글턴을 활용한 예로는 무상태 객체(stateless) 또는 설계상 유일해야 하는 시스템 컴포넌트 등



### 싱글톤의 단점

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트는 테스트하기가 어려워질 수 있다고 합니다.

좋은 객체 지향을 설계하기 위한 원칙 중에서 의존성을 인터페이스로 설계하면 실제 구현 클래스의 내용이 변경되어도 이를 사용한 코드는 영향을 받지 않게 됩니다. 하지만 싱글톤을 사용하는 경우 대부분 인터페이스가 아닌 구현 클래스의 객체를 미리 생성해놓고 정적 팩토리 메소드를 이용하여 구현하게 됩니다. 이는 SOLID 원칙을 위반하게 될 수 있고 동시에 싱글톤을 사용하는 곳과 싱글톤 객체 사이에 의존성이 생기게 됩니다. 이렇게 되면 의존성으로 인하여 결합도가 생기게 되고 이로 인하여 테스트의 어려움이 생길 수 있습니다.



## 싱글턴을 만드는 방식

- `public static final` 필드 방식의 싱글턴
- 정적 팩토리 방식의 싱글턴
- 열거 타입 방식의 싱글턴



### `public static final` 필드 방식의 싱글턴

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
      	//...
    }
}
```

- private 생성자는 public static final 필드인 `Elvis.INSTANCE` 를 초기화할 때 딱 한 번만 호출됩니다. 
- public static 필드가 final 이니 절대로 다른 객체를 참조할 수 없습니다.
- 해당 클래스가 싱글턴임이 API에 명백히 드러납니다.
- public 또는 proctected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장됩니다.



### 정적 팩토리 메소드

```java
public class Adel {
    
    private static final Adel INSTANCE = new Adel();
    
    private Adel() {
      //....
    }
    
    public static Adel getInstance() {
        return INSTANCE;
    }
}
```

- `Adel.getInstance` 는 항상 같은 객체의 참조를 반환하므로 제2의 Adel 인스턴스는 절대 만들어지지 않습니다.
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있습니다.
- 정적 팩토리의 메소드 참조를 공급자(`supplier`)로 사용할 수 있습니다. 
  - `Elvis:getInstance()` -> `Supplier<Elvis>`  




### 열거 타입 방식의 싱글턴

```java
public enum Queen {

    INSTANCE;
}
```

- public 필드 방식과 비슷하지만, 더 간결합니다.
- 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법입니다.
- 만드려는 싱글턴이 enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없습니다.



## 정리

- 자바를 이용한 싱글턴 방식에서는 열거 타입 방식의 싱글턴을 사용합니다.
  - 스프링 프레임워크의 싱글턴과는 다릅니다.