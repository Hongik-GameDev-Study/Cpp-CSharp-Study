# C#에서의 상등

---

# 기본적인 상등

## ==연산자와 ≠연산자

기본적으로 간단히 연산자로 객체를 비교할 수 있다.

그럼 **값 형식은 값 상등비교**를 **참조 형식은 참조 상등비교**로 비교한다.

하지만 이를 우리가 원하는 동작을 하도록 **중복적재 Overloading** 할 수 있다.

```csharp
public class MyData
{
    public int X;

    public static bool operator==(MyData rhs, MyData lhs)
    {
        return rhs.X == lhs.X;
    }

    public static bool operator !=(MyData rhs, MyData lhs)
    {
        return !(rhs.X == lhs.X);
    }
}

public class Test
{
    public static void Main()
    {
        var d1 = new MyData() { X = 5 };
        var d2 = new MyData() { X = 5 };
        Console.WriteLine(d1 == d2); // True
    }
}
```

## Object에서 상속받은 것들

클래스를 만들면 아무런 상속을 받지 않아도 기본적으로 `Object` 클래스에서 상속받는 것들이 있다.

그 중에 `Equals`라는 메서드가 있다.

```csharp
public class MyData
{
    public int X;

    public override bool Equals(object obj)
    {
        if (obj == null)
            return false;

        if (!(obj is MyData))
            return false;

        return X == ((MyData)obj).X;
    }
}

public class Test
{
    public static void Main()
    {
        var d1 = new MyData() { X = 5 };
        var d2 = new MyData() { X = 5 };
        Console.WriteLine(d1.Equals(d2)); // True
    }
}
```

참조 형식이지만 `Equals`를 재정의하여 값 비교처럼 동작하게 한 예시이다.

- **??? ==연산자가 따로 있는데 Equals가 있는 이유를 모르겠어.**
    
    ```csharp
    public class Enemy
    {
    
        public int health;
    
        public static bool operator==(Enemy rhs, Enemy lhs)
        {
            return rhs.health == lhs.health;
        }
    
        public static bool operator !=(Enemy rhs, Enemy lhs)
        {
            return rhs.health != lhs.health;
        }
    
        public override bool Equals(object obj)
        {
            if (!(obj is Enemy)) return false;
            return ((Enemy)obj).health == health;
        }
    }
    
    public class Troll : Enemy
    {
    
        public int attack;
    
        public static bool operator ==(Troll rhs, Troll lhs)
        {
            return rhs.health == lhs.health && rhs.attack == lhs.attack;
        }
    
        public static bool operator !=(Troll rhs, Troll lhs)
        {
            return rhs.health != lhs.health && rhs.attack != lhs.attack;
        }
    
        public override bool Equals(object obj)
        {
            if (!(obj is Troll)) return false;
            return ((Troll)obj).health == health && ((Troll)obj).attack == attack;
        }
    }
    
    public class Bat : Enemy
    {
        public int speed;
    
        public static bool operator ==(Bat rhs, Bat lhs)
        {
            return rhs.health == lhs.health && rhs.speed == lhs.speed;
        }
    
        public static bool operator !=(Bat rhs, Bat lhs)
        {
            return rhs.health != lhs.health && rhs.speed != lhs.speed;
        }
    
        public override bool Equals(object obj)
        {
            if (!(obj is Bat)) return false;
            return ((Bat)obj).health == health && ((Bat)obj).speed == speed;
        }
    }
    ```
    

## 정적 메서드들

위에서 본 `Object`의 가상 `Equals` 메서드는 제한점이 하나 있다.

`null`에다가는 하지 못한다는 단점인데 이를 위해 `object.Equals`라는 정적 메서드가 존재한다.

```csharp
public class MyData
{
    public int X;

    public override bool Equals(object obj)
    {
        if (obj == null)
            return false;

        if (!(obj is MyData))
            return false;

        return X == ((MyData)obj).X;
    }

    public static bool operator==(MyData rhs, MyData lhs)
    {
        return rhs.X == lhs.X;
    }

    public static bool operator !=(MyData rhs, MyData lhs)
    {
        return !(rhs.X == lhs.X);
    }
}

public class Test
{
    public static void Main()
    {
        var d1 = new MyData() { X = 5 };
        var d2 = new MyData() { X = 5 };
        Console.WriteLine(object.Equals(d1, d2)); // True
    }
}
```

내부적으로 가상 메서드 `Equals`를 호출하기 때문에 정적 메서드 `Equals`를 호출하더라도 재정의한 `Equals`가 호출된다.

그리고 재정의했더라도 참조 상등 비교를 하고 싶은 경우를 위한 `Object.ReferenceEquals` 메서드가 존재한다.

```csharp
public class Test
{
    public static void Main()
    {
        var d1 = new MyData() { X = 5 };
        var d2 = new MyData() { X = 5 };
        Console.WriteLine(object.Equals(d1, d2)); // True
        d1 = null;
        Console.WriteLine(object.Equals(d1, d2)); // False
        d1 = new MyData() { X = 5 };
        Console.WriteLine(object.ReferenceEquals(d1, d2)); // False
    }
}
```

## 상등과 비교를 위한 인터페이스

이전의 예시로는 클래스를 들었지만, 값 형식의 피연산자들에 대해 `object.Equals`를 호출하면 반드시 박싱이 적용된다. 박싱은 값 비싼 연산이므로 이를 피하기위해 C#2.0에는 `IEquatable<T>` 인터페이스가 도입되었다.

```csharp
public struct MyData : IEquatable<MyData>
{
    public int X;

    public bool Equals(MyData other)
    {
        return X == other.X;
    }
}
```

실제로 .NET Framework의 기본 형식들은 대부분이 이를 상속받아 구현한다.

## GetHashCode

번외로 `GetHashCode` 메서드도 `System.Object`에서 상속함.

이는 해시테이블 컬렉션에서 사용하게 되는데, `Equals`나 `==`연산자를 재정의 혹은 중복적재하게 되면 `GetHashCode`도 재정의해주는 것이 바람직함.

`GetHashCode`를 재정의할 때 따라야 할 규칙은 다음과 같다.

- `Equals`가 `true`를 돌려주는 두 객체에 대해 `GetHashCode`도 반드시 `true`를 돌려줘야 한다. (둘 중 하나를 재정의한다면 나머지 하나를 재정의해야하는 이유)
- 예외를 던지면 안 된다.
- 도중에 변하지 않은 같은 객체에 되풀이해서 호출되었을 때 같은 값을 돌려줘야 한다.

재정의 안해줘도 괜찮을 수 있지만 프로그램이 커지면 어디서(프로그래머가 작성했거나 .NET Framework에서) `GetHashCode`를 사용할지 모르기때문에 재정의해주는 것이 좋다.

# 컬렉션에서의 상등

## 상등 및 순서 비교 플러그인

- `Equals`와 `GetHashCode`가 의미 있는 결과를 돌려주는 형식은 `Dictionary`나 `Hashtable`에서 키로 쓸 수 있다.
- `IComparable`/`IComparable<T>`를 구현하는 형식은 임의의 정렬된 사전이나 목록에서 키로 쓰일 수 있다.

키로 정렬을 하고 싶지만 `string`인 사전에서 대소문자 구분없이 검색을 하고 싶을 경우,

어떤 고객 목록을 고객의 이름이 아닌 고객의 우편번호 순으로 정렬하고 싶은 경우에 사용할 수 있도록 .NET Framework는 일단의 '플러그인' 프로토콜들을 제공한다.

이 플러그인 프로토콜들은 다음 두 가지 역할을 한다.

- 기본 방식과는 다른 방식의 상등 비교 또는 순서 비교를 적용할 수 있게 한다.
- 상등 비교나 순서 비교 능력이 갖추어져 있지 않은 형식을 사전이나 정렬된 목록의 키 형식으로 사용할 수 있게 한다.

다음은 플러그인 프로토콜을 구성하는 인터페이스들이다.

- `IEqualityComparer`와 `IEqualityComparer<T>`
    - 플러그인 상등 비교와 해싱 기능을 담당한다.
    - `Hashtable`과 `Dictionary`가 가능해진다.

- `IComparer`와 `IComparer<T>`
    - 플러그인 순서 비교 기능을 담당한다.
    - 정렬된 사전과 컬렉션, 그리고 `Array.Sort`가 가능해진다.

.NET Framework 4.0에는 `IStructrualEquatable`과 `IStructrualComparable`이라는 새로운 인터페이스도 도입되었고 이들은 클래스와 배열에 대한 구조적 비교 능력을 제공한다.

### IEqualityComparer 인터페이스와 EqualityComparer 클래스

이들은 기본 방식 이외의 상등 비교와 해시 계산 구현을 위한 상등 비교자를 만드는데 쓰인다.

이는 `Dictionary` 클래스와 `Hashtable` 클래스에서 유용하다.

`IEqualityComparer` 인터페이스는 해시테이블 기반 사전들의 요구 사항에 답한다.

- 그 키와 같은 키가 존재하는가?
- 그 키의 정수 해시코드는 무엇인가?

```csharp
public interface IEqualityComparer<T>
{
	bool Equals (T x, T y);
	int GetHashCode(T obj);
}

public interface IEqualityComparer
{
	bool Equals (object x, object y);
	int GetHashCode (object obj);
}
```

상등 비교자를 작성할 때에는 이 두 인터페이스 중 하나 또는 둘 다를 구현한다.

그런데 두 인터페이스를 구현하는 것은 다소 지루한 일이다. 그 대신 다음과 같이 정의된 추상 클래스 `EqualityComparer`를 상속하는 방법도 있다.

```csharp
public abstract class EqualityComparer<T> : IEqualityComparer, IEqualityComparer<T>
{
	public abstract bool Equals (T x, T y);
	public abstract int GetHashCode (T obj);

	bool IEqualityComparer.Equals (object x, object y);
	int IEqualityComparer.GetHashCode (object obj);

	public static EqualityCompaerer<T> Default { get; }
}
```

`EqualityComparer` 클래스는 두 인터페이스 모두 구현하고 있으므로 상등 비교 클래스 작성자는 그냥 추상 메서드 두 개만 재정의하면 된다.

`Equals`와 `GetHashCode`의 의미론은 6장에서 설명한 `object.Equals` 및 `object.GetHashCode`와 동일한 규칙을 따른다.

다음은 고객을 뜻하는 `Customer` 클래스의 두 고객의 상등을 판정하는 상등 비교자의 예이다.

고객 클래스에는 성과 이름을 뜻하는 두 필드가 있으며, 상등 비교자는 성과 이름 둘 다 같을 때에만 두 고객이 상등이라고 판정한다.

```csharp
public class Customer
{
	public string LastName;
	public string FirstName;

	public Customer(string last, string first)
	{
		LastName = last;
		FirstName = first;
	}
}

public class LastFirstEqComparer : EqualityComparer<Customer>
{
	public override bool Equals (Customer x, Customer y)
	{
		return x.LastName == y.LastName && x.FirstName == y.FirstName;
	}

	public override int GetHashCode (Customer obj)
	{
		return (obj.LastName + ";" + obj.FirstName).GetHashCode();
	}
}
```

이 클래스의 작동 방식을 보여주는 예로 다음과 같이 이름이 같은 고객 둘을 생성한다.

```csharp
Customer c1 = new Customer("Bloggs", "Joe");
Customer c2 = new Customer("Bloggs", "Joe");
```

`object.Equals`를 재정하지 않았기 때문에, 이 둘에 대해 보통의 참조 상등 의미론이 적용된다.

```csharp
Console.WriteLine(c1 == c2); // False
Console.WriteLine(c1.Equals(c2)); // False
```

이 고객들을 상등 비교자를 지정하지 않고 `Dictionary`에 사용하면 역시 동일한 참조 상등 의미론이 적용된다.

```csharp
var d = new Dictionary<Customer, string>();
d[c1] = "Joe";
Console.WriteLine(d.ContainsKey(c2)); // False
```

그러나 커스텀 상등 비교자를 지정하면 두 고객이 같다는 결과를 얻을 수 있다.

```csharp
var eqComparer = new LastFirstEqComparer();
var d = new Dictionary<Customer, string>(**eqComparer**);
d[c1] = "Joe"
Console.WriteLine(d.ContainsKey(c2)); // True
```