## private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라
JDK 1.5 버전 이전에는 private 생성자를 이용한 싱글턴 패턴이 사용됐다.
하지만 이 싱글턴 패턴에는 두가지 문제가 있는데
하나는 리플렉션을 통해 생성자에 접근할 수 있는 문제이고 다른 하나는 싱글턴 객체가 serializable 을 상속받게 되면 역직렬화 될 때마다 새 객체가 생성되는 문제 이다.

첫 번째 문제는 생성자에서 두번째 객체 생성 시도시 오류를 던지게 해서 방어할 수 있으며 두번째 문제는 아래 함수를 추가해 해결해 주어야한다.

```java
private Object readResolve() 
{
	return INSTANCE;
}
```

**참고** 리플렉션을 통해 private 생성자 접근하기
```java

// 리플렉션을 통해 private 생성자 접근하기
import java.lang.reflect.Constructor;

public class PrivateInvoker
{
	public static void main(String[] args) throws Exception
	{
		Constructor<?> con = Private.class.getDeclaredConstructors(0[0];
		con.setAccessible(true);
		Private p = (Private) con.newInstance();
	}
}

class Private
{
	private Private()
	{
		System.out.println("Hello");
	}
}
```
싱글턴 객체를 선언할 때 class가 아닌 enum을 이용하면 위에 제시한 두가지 문제를 간단히 해결할 수 있다.
```java
public enum Elvis
{
	INSTANCE;
	
	public void leaveTheBuilding(){...}
}
```

하지만 객체 생성 시점에 다른 객체를 참조하기 힘들다는 문제가 있어 최근에는 LayHolder 라고 불리는 아래 방법이 더 선호된다.
```java
public class Singleton 
{
	private Singleton() {}
	public static Singleton getInstance() 
	{
		return LazyHolder.INSTANCE;
	}
  
	private static class LazyHolder 
	{
		private static final Singleton INSTANCE = new Singleton();  
	}
}
```