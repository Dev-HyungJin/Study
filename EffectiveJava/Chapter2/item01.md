## 1 생성자 대신 정적 팩터리 메서드를 사용할 수 없는지 생각해보라
public 으로 선언한 생성자 대신 public으로 선언된 정적 팩터리 메서드를 추가하는 것

### 1.1 장점
#### 1.1.1 생성자와 달리 이름을 가질 수 있다.
여러가지 시그니처를 가진 생성자를 가진 클래스가 있다면 API 사용자가 실수하기 쉬우며 설명서를 읽지 않으면 각 생성자의 파라미터가 어떤 의미를 갖는지 알 수 없다.
``` java
// 일반적인 생성자
public BigInteger(boolean probablePrime){
	...
}

// 생성자의 patameter 들을 확인해봐야 한다.
BigInteger instance = new BigInteger(true);

```
``` java
// 정적 팩터리 메서드 사용
public static BigInteger probablePrime(){
	...
	return new BigInteger(...);
}

// 메서드 이름으로 유추 할 수 있다.
BigInteger instance = BigInteger.probablePrime();

```
#### 1.1.2 호출할 때마다 새 객체를 생성할 필요는 없다.
변경 불가능 클래스라면 이미 만들어 둔 객체를 활용할 수 도 있고 만든 객체를 캐시해두고 재사용하게 할 수 있다.
정적 팩터리 메서드를 사용하는 클래스는 **개체 통제 클래스(instance-controlled class)**다.
개체 통제 클래스는 *어떤 시점에 어떤 객체가 얼마나 존재할지* 정밀하게 제어할 수 있는 클래스를 말하며 아래 가지 장점을 갖는다.
> - 개체 수를 제어해 싱글턴 패턴을 유도할 수 있다.
> - 객체 생성이 불가능한 클래스를 만들 수 있다.
> - 변경이 불가능한 클래스의 경우 두개의 같은 객체가 존재하지 못하도록 할 수 있다.<br>이 경우 equals(Object) 대신  == 연산을 사용할 수 있어 성능이 향상된다.
#### 1.1.3 반환값 자료형의 하위 자료형을 반환할 수 있다.
이 특징을 활용해 public으로 선언되지 않은 하위 클래스의 객체를 반환하는 API를 만들 수 있는데 이는 구현 세부사항을 감출 수 있어 간결한 API를 만들 수 있게 해준다.
```java
public interface Service
{
	// Logic
}
public class DefaultService implements Service
{
	...
}
public class FirstService implements Service
{
	...
}
public class SecondService implements Service
{
	...
}
```

```java
// Provider 제공여부는 옵션으로 반드시 있을 필요는 없다.
public interface Provider
{
	Service newService(); 
}
public class DefaultProvider implements Provider
{
	@override
	Service newServcie(){return new DefaultService();}
}
public class FirstProvider implements Provider
{
	@override
	Service newServcie(){return new FirstService();}
}
public class SecondProvider implements Provider
{
	@override
	Service newServcie(){return new SecondService();}
}
```
```java
public class Services
{
	private Map<String, Provider> providers;
	
	public registerDefaultProvider(Provider prov) {...}
	public registerProvider(String name, Provider prov) {...}
	
	public Service newInstance(){...}	// return defualt provider
	public Service newInstance(String name) {...}
}
```
```java
public calss Test
{
	public static void main(String[] args)
	{
		Services.registerDefaultProvider(new DefaultProvider());
		Services.registerProvider("case1", new FirstProvider());
		Services.registerProvider("case2", new SecondProvider());
		
		Service s1 = Services.newInstance();
		Service s1 = Services.newInstance("case2");
		...
		// 각 Provider 내부에서 반환하는 Service의 종류와 무관하게
		// Service insterface만 알면 사용할 수 있다.
	}
}
```
#### 1.1.4 ~~형인자 자료형을 만들 때 편하다.~~
1.7 버전부터 생성자 호출시에도 자료형 유추를 사용할 수 있어 필요 없어졌다.
```java
// () 앞에 <>를 붙이지 않으면 Warning이 발생한다.
Map<String, List<String>> m = new HashMap<>();
```
~~아래 처럼 자료형을 중복으로 사용하는 경우 형인자가 늘어나면서 코드가 길고 복잡해진다.~~
```java
Map<String, List<String>> m = new HashMap<String, List<String>>();
```
~~이때 아래처럼 제네릭 정적 팩터리 메서드가 제공되면 코드를 간략하게 줄일 수 있다.~~
```java
public static <K, V> HashMap<K, V> newInstance()
{
	return new HashMap<K,V>();
}

Map<String, List<String>> m = new HashMap.newInstance();
```
### 1.2 단점
#### 1.2.1 정적 팩터리 메서드만 있는 클래스를 만들면 하위 클래스를 만들 수 없다.
public이나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다.
논쟁의 여지가 있지만 구성(composision) 기법을 장려한다는 이유에서 장점으로 보는 사람도 있다.
#### 1.2.2 정적 팩터리 메서드가 다른 정적 메서드와 확연히 구분되지 않는다.
생성자와 달리 정적 팩터리 매서드는 일반 정적 메서드와 구분되지 않아 사용자가 구분하기 힘든 문제가 있어 이름을 지을 때 주의하는 수 밖에 없다.
아래 주로 사용하는 메서드 이름을 참고하자.
`ValueOf, Of, getInstance, newInstance, getType, newType `
