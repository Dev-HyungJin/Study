## 4 객체 생성을 막을 때는 private 생성자를 사용하라
정적 메서드나 필드만 모은 클래스를 만들고 싶을 때 private 생성자를 넣어 객체 생성을 방지하자
```java
public class UtilityClass
{
	private UtilityClass()
	{
		// AssertionError가 반드시 필요하지는 않지만 
		// 클래스 내부에서 실수로 생성하는 것 까지 막기 위해 추가한다.
		throw new AssertionError();
	}
	...
}
```

## 5 불필요한 객체는 만들지 말라

아래 코드에서 "stringette"는 자체로 String객체기 때문에 다시 생성할 필요가 없다.
특히 이 코드가 반복문이나 빈번히 호출되는 메서드 안에 있다면 불필요한 String 객체가 엄청나게 만들어질 수 있다.
``` java
String s = new String("stringette");
```
따라서 아래 처럼 String 객체를 바로 할당해 사용하는 것이 좋다.
아래 같은 코드는 같은 VM에서 실행될 때 모두 같은 객체를 사용하게 한다.
```java
String s = "stringette";
```
변경 불가능 객체의 경우 새 객체를 생성하기 보다 변경 불가 객체를 반환하는 함수를 사용하는 것이 좋다.
```java
boolean b = new Boolean("True");

// 아래 방법이 더 좋다.
boolean b = Boolean.valueOf("True");
```

변경 가능 객체도 변경할 필요가 없다면 재사용이 가능하다.
예를 들어 아래 isBabyBoomer 메서드는 호출할 때마다 불필요한 Calendar, Date, TimeZone 객체를 만들어 댄다.
```java
public class Person
{
	private final Date birthDate;
	
	
	public boolean isBabyBoomer()
	{
		Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
		gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
		Date boomStart = gmtCal.GetTime();
		gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
		Date boomEnd = gmtCal.GetTime();
		
		return birthDate.CompareTo(boomStart) >= 0 &&
			birthDate.CompareTo(boomEnd) < 0;
	}
}
```
위처럼 반복적으로 사용되는 데이터들은 정적 초기화 블록을 이용해 클래스가 초기화 될 때 한번만 초기화되게 만들면 성능을 크게 개선할 수 있다.
그러나 성능이 크게 개선될 수 있는 이유는 Calendar객체의 생성 비용이 비싸기 때문이며 일반적으로는 큰 성능개선을 기대하기는 힘들다.

```java
public class Person
{
	private final Date birthDate;
	
	private static final Date BOOM_START; 
	private static final Date BOOM_END;
	
	static
	{
		Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
		gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
		BOOM_START = gmtCal.GetTime();
		gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
		BOOM_END = gmtCal.GetTime();
	}
	
	public boolean isBabyBoomer()
	{
		return birthDate.CompareTo(BOOM_START) >= 0 &&
			birthDate.CompareTo(BOOM_END) < 0;
	}
}
```

어떤 상황에서는 객체 재사용 여부가 분명하지 않을 수 있는데 Adaptor처럼 실제 기능 수행은 후면객체(backing object)에 위임하고 
후면객체의 인터페이스만 제공하는 객체의 경우 객체에 대한 Adaptor를 여러개 만들 필요가 없다.

 JDK 1.5부터는 자동 객체화 (autoboxing)이라는 것이 생겨 쓸데 없는 객체가 생성될 가능성이 높아졌다.
 자동객체화는 자바의 기본 자료형(primitive type)과 객체 표현형을 섞어 사용할 수 있도록 해주는데 아래 같은 코드에서 성능저하를 가져올 수 있으므로 주의해야 한다.
 
```java
 public static void main(String[] args)
 {
 	Long sum = 0L;
 	for(long i = 0; i < Integer.MAX_VALUE; i++)
 	{
 		// Long은 long 타입의 표현형이기 때문에 불필요한 Long객체가 Loop 수 만큼 생성된다.
 		sum += 1;
 	}
 	System.out.println(sum);
 }
```

 위같은 문제를 피하기 위해서는 **객체 표현형 대신 기본 자료형을 사용**하고 생각지도 못한 자동 객체화가 발생하지 않도록 조심해야한다.
 
 위에서 말하는 것이 객체 생성비용을 피하기 위해 객체 사용을 하지 말하는 것은 아니다. 
 생성자 안에서 하는 일이 작고 명확하다면 객체 생성과 반환은 신속하게 이루어지기 때문에 객체를 통해 코드의 명확성과 단순성을 높일 수 있다면 일반적으로는 만드는 것이 좋다.
 
 마찬가지로 객체 풀(Object Pool) 또한 코드가 어려워지고 메모리 요구량이 증가하며 성능도 떨어지기 때문에 객체 생성비용이 극단적으로 높지 않다면 피해야 한다. (최신 JVM은 가벼운 객체에 대해 객체 풀보다 월등히 뛰어난 성능을 보여준다.)
 
 
 