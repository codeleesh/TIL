# [아이템14] Comparable을 구현할지 고려하라

이 내용은 `이펙티브 자바 Effective Java 3/E` 를 읽으면서 정리한 내용을 포함하고 있습니다.



`Comparable` 인터페이스의 유일무이한 메서드인 `compareTo` 를 알아보도록 하겠습니다. `compareTo` 는 Object의 메서드가 아닙니다. equals와 거의 비슷하나 2가지가 다릅니다. 

첫째는, compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있습니다.

둘째는, 제네릭 합니다.

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음으 뜻합니다. 그래서 Comparable을 구현한 객체들의 배열은 손쉽게 정렬할 수 있습니다.

```java
Arrays.sort(a);
```



아래 코드는 중복을 제거하고 알파벳순으로 출력합니다. String이 Comparable을 구현한 덕분입니다.

 ```java
 public class WordList {
 
 	public static void main(String[] args) {
 	
 		Set<String> s = new TreeSet<>();
 		Collections.addAll(s, args);
 		System.out.println(s);
 	}
 }
 ```



## compareTo 규약

compareTo 메서드의 일반 규약은 equals의 규약과 비슷합니다.

- 이 객체와 주어진 객체의 순서를 비교합니다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환합니다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던집니다.
- Comparable을 구현한 클래스는 모든 x, y에 대해 sign(x.compareTo(y)) == -sign(y.compareTo(x)) 여야 합니다.
- Comparable을 구현한 클래스는 추이성을 보장해야 합니다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면 x.compare(z) > 0 입니다.
- Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0 이면 sign(x.compareTo(z)) == sign(y.compareTo(z)) 입니다.
- 이번 권고는 필수는 아니지만 꼭 지키는 것이 좋습니다. (x.compareTo(y) == 0) == (x.equals(y))여야 합니다.



첫 번째 규약은 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 이야기입니다.

- 첫 번째 객체가 두 번째 객체보다 작으면, 두 번째가 첫 번째보다 커야 합니다.
- 첫 번째가 두 번째와 크기가 같다면, 두 번째는 첫 번째보다 같아야 합니다.
- 첫 번째가 두 번째보다 크면, 두 번째는 첫 번째보다 작아야 합니다.

두 번째 규약은 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다는 뜻입니다.

세 번째 규약은 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다는 뜻입니다.



## compareTo 작성 요령

Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일 타임에 정해집니다. 입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻입니다. 인수의 타입이 잘못되었다면 컴파일 자체가 되지 않습니다.

또한 null을 인수로 넣어 호출하면 NullPointerException을 던져야 합니다. 

compareTo 메서드는 각 필드가 동치인지를 비교하는 것이 아니라 그 순서를 비교합니다. 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출해야 합니다. Compareble을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용합니다. 비교자는 직접 만들거나 자바가 제공하는 것 중에 골라서 사용하면 됩니다.



자바가 제공하는 비교자

```
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {

	public int compareTo(CaseInsensitiveString cis) {
		
		return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
	}
}
```

- compareTo 메서드에서 관계 연산자<와> 를 사용하는 방식은 거추장스럽고 오류를 유발하니 이제는 추천하지 않습니다.



클래스에 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해집니다. 가장 핵심적인 필드부터 비교해 나가고, 비교 결과가 0이 아니라면, 즉 순서가 결정되면 거기서 끝입니다. 가장 핵심이 되는 필드가 똑같다면, 똑같지 않은 필드를 찾을 때까지 그다음으로 중요한 필드를 비교해나갑니다. 

```java
public int compareTo(PhoneNumber pn) {

	int result = Short.compare(areaCode, pn.areaCode);
	if (result == 0) {
		result = Short.compare(prefix, pn.prefix);
		if (result == 0) {
			result = Short.compare(lineNum, pn.lineNum);
		}
	}
	return result;
}
```



자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 체이닝 방식으로 비교자를 생성할 수 있습니다.





## 추천하는 방식

정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
		return Integer.compare.(o1.hashCode(), o2.hashCode());
	}
}
```



비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```



## 정리

- 순서를 고려해야 하는 값 클래스를 작성한다면 `Comparable` 인터페이스를 구현해야 하며, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 합니다.
- compareTo 메서드에서 필드의 값을 비교할 때 <와> 연산자는 쓰지 말아야 합니다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용해야 합니다.
