## 생성자 인자가 많을 때는 Builder 패턴 적용을 고려하라
생성자의 인자가 많을 때 일반적으로 **점증적 생성자 패턴**을 적용한다.
하지만 점증적 생성자 패턴은 인자수가 늘어남에 따라 클라이언트 **코드를 작성하기 어려워지고 읽기 어려운 코드가 된다.**
```java
public calss Student
{
	public Student(int age, int gender){}
	public Student(int age, int gender, String name)
	{
		this(age, gender);
	}
	public Student(int age, int gender, String name, int grade)
	{
		this(age, gender, name);
	}
	
	// 불필요한 인자가 생기게 된다.
	Student student = new Student(10, 1, "", 4);
}
```

다른 대안으로 **자바빈 패턴**을 적용할 수 있다.
이 패턴의 문제점은 **순간적으로 생성자의 일관성이 깨질 수 있고 변경 불가능한 객체를 만들 수 없다**는 것이다.
```java
public calss Student
{
	public Student(int age, int gender){}
	public void SetName(String name) {...};
	public void SetGrade(int grade) {...};
	
	// 불필요한 인자가 생기게 된다.
	Student student = new Student(10, 1);
	student.SetGrade(4);
}
```

또 다른 대안으로 **빌더 패턴**을 적용해 볼 수 있다.
```java
public calss Student
{
	public Student(Builder builder){...}
	
	public static calss Builder
	{
		public Builder(){...}
		public void Age(int age){...; return this;}
		public void Gender(int gender){...; return this;}
		public void Name(int name){...; return this;}
		public void Grade(int grade){...; return this;}
		
		public Student Build()
		{
			return Student(this);
		}
	}
}

//example
Student student = Student.Builder().Age(10).Name("Tom").Build();

```

빌더 패턴은 인자가 많은 생성자나 정적 팩터리가 필요한 클래스를 설계할 때, 특히 **대부분의 인자가 선택적 인자인 상황**에서 유용하다.
빌더 패턴은 빌더를 생성한 후 실제 생성자를 호출하게되는 오버헤드가 있지만 객체 생성의 오버헤드는 극단적으로 고성능이 필요한 상황이 아니면 그 영향이 크지 않고
빌더를 추가하면서 코드 작성량이 많아지는 것이 더 큰 단점으로 지적된다.