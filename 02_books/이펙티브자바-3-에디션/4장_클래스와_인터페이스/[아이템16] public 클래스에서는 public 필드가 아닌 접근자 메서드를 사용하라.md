# [아이템16] public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

이 내용은 `이펙티브 자바 Effective Java 3/E` 를 읽으면서 정리한 내용을 포함하고 있습니다.

인스턴스 필드들만 모아놓은 값 객체를 작성하는 경우가 종종 있습니다.



## 인스턴스 필드들만 모아놓은 객체

```java
class Point {

	public double x;
	public double y;
}
```

이런 클래스는 단점은 다음과 같습니다.

- 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못합니다. 
- API를 수정하지 않고는 내부 표현을 바꿀 수 없습니다. 
- 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없습니다. 



철저한 객체 지향 개발자는 이런 클래스를 상당히 싫어해서 필드들을 모두 private으로 바꾸고 public 접근자(getter)를 추가합니다.

```java
class Point {

	public double x;
	public double y;
	
	public Point(double x, double y) {
	
		this.x = x;
		this.y = y;
	}
	
	public double getX() { return x; }
	public double getY() { return y; }
	
	public void setX(double x) { this.x = x; }
	public void setY(double y) { this.y = y; }
}
```

public 클래스에서라면 이 방식이 확실히 맞습니다. 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀수 있는 유연성을 얻을 수 있습니다.

package-private 클래스 또는 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없습니다. 



## public 클래스의 필드 직접 노출

public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있습니다. 대표적인 예가 `java.awt.package` 패키지의 Point와 Dimension 클래스입니다. 

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각은 아닙니다. API를 변경하지않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전히 존재합니다. 



## 정리

- public 클래스는 절대 가변 필드를 직접 노출해서는 안 됩니다.
- 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없습니다.
- package-private 클래스 또는 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 수 있습니다.