## 6 유효기간이 지난 객체 참조는 폐기하라
아래 코드는 특별히 문제가 없어 보이지만 자세히 살펴보면 메모리 누수(memory leak)문제를 찾을 수 있다.
```java
public class Stack
{
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY=16;

	public Stack()
	{
		ements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	
	public void push(Object e)
	{
		ensureCapacity();
		element[size++] = e;
	}

	public Object pop()
	{
		if(size == 0)
			throw new EmptyStackException();
		return elements[--size];
	}
	
	private void ensureCapacity()
	{
		if(elements.length == size)
		{
			elements = Arrays.copyOf(elements, 2*size+1);
		}
	}
}
```
pop 함수에서 값을 반환 한 뒤 elements의 할당된 객체를 null 로 초기화하지 않기 때문에 만기 참조(obsolete reference)가 제거되지 않아 메모리 누수가 발생한다.
GC를 사용하는 언어에서 발생하는 메모리 누수(의도치 않은 객체 보유 ^unintentional object retention^)는 찾아내기가 어렵고 해당 객체가 참조하는 모든 객체가 함께 수집에서 제외되기 때문에 생각보다 문제가 심각하게 나타날 수 있다.

```java
public Object pop()
{
	if(size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null;
	return result;
}
```
해당 문제는 위처럼 간단히 해결할 수 있지만 모든 객체 참조를 null로 초기화 해야된다는 말은 아니니 주의 해야한다.
필요하지 않은 경우에도 모든 참조를 null로 초기화하면 코드만 난잡해질 수 있기 때문에 예외적으로 메모리 누수가 발생할 수 있는 경우에만 null로 초기화 해주는 것이 좋다.
일반적으로 **자체적으로 관리하는 메모리가 있는 클래스**를 만들 때 메모리 누수가 발생하지 않도록 주의해야 하며 더 이상 사용하지 않는 객체 참조는 null로 바꿔 주어야한다.
**캐시(cache)도 메모리 누수가 흔히 발생하는 장소다.** 이 경우 WeakHashMap을 가지고 캐시를 구현하면 캐시 외부에서 참조가 사라질때 캐시 값도 사라지기 때문에 효율적으로 관리할 수 있다. 
하지만 일반 적으로 캐시는 외부 참조 보다는 보관된 기간에 따라 수명이 결정되기 때문에 후면 스레드(background thread)를 사용해 제거하거나 LinkedHashMap 클래스와 removeEldestEntry 함수를 사용해 캐시에 새로운 항목을 추가할 때 오래된 항목을 제거할 수 도있다.
또 메모리 누수가 흔히 발견되는 곳은 **리스너(listener) 등의 역호출자(callback)이다.** 역호출자를 사용하는 API를 사용 후 클라이언트가 명시적으로 제거하지 않으면 메모리에 점유된 상태로 남아있기 때문이다. 위에서 언급했던 WeakHashMap을 사용해 약한 참조(weak reference)를 만드는 것이 하나의 해결책이 될 수 있다.
메모리 누수는 일반적으로 뚜렷한 오류로 나타나지 않아 수년간 발견되지 않는 경우가 빈번하다 따라서 이런 문제가 언제나 발생할 수 있다는 것을 인지하고 사전에 방지 대책을 세워두는 것이 바람직하다.