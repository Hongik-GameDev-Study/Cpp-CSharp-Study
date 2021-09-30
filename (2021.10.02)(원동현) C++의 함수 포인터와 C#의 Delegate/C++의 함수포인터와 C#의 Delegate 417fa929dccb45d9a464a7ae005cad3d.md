# C++의 함수포인터와 C#의 Delegate

> # C++

## 함수 포인터

```cpp
int Add(int a, int b) { return a + b; }
```

```cpp
int(*f)(int, int) = Add;
```

위가 일반적인 문법

```cpp
int(*f)(int, int) = &Add;
```

이렇게도 가능.

사실 `&`없이 하는 것은 C언어와의 호환성때문이라고 합니다.

로키스님은 함수 객체(Functor)랑 통일하기위해서 `&`를 쓰는 문법을 선호한다고 합니다.

```cpp
int(*f)(int, int) { Add };
```

이렇게도 가능.

```cpp
f(1,3);
(*f)(1, 4);
```

호출하는 방법도 두가지.

이것도 C언어와의 호환성때문입니다.

함수에 함수 포인터 전달은 이런식으로 사용할 수 있습니다.

```cpp
class Item
{
public:
    Item() : _itemId(0), _rarity(0), _ownerId(0)
    {
    }

public:
    int _itemId;
    int _rarity;
    int _ownerId;
};

Item* FindItem(Item items[], int itemCount, **bool(*selector)(Item* item)**)
{
    for (int i = 0; i < itemCount; i++)
    {
        Item* item = &item[i];

        if (selector(item))
            return item;
    }

    return nullptr;
}

bool IsRareItem(Item* item)
{
    return item->_rarity > 2;
}

int main()
{
    Item items[10] = {};
    items[3]._rarity = 4;
    Item* itm = FindItem(items, 10, IsRareItem);

    return 0;
}
```

`selector`로 어떤 Item을 찾는 `FindItem` 함수의 조건을 호출자에서 정해줄 수 있게 됩니다.

강의에서는 가독성때문인지 `typedef`를 이용해서 조금 바꿔주네요.

```cpp
**typedef bool(ITEM_SELECTOR)(Item*);**

Item* FindItem(Item items[], int itemCount, **ITEM_SELECTOR* selector**)
{
    for (int i = 0; i < itemCount; i++)
    {
        Item* item = &item[i];

        if (selector(item))
            return item;
    }

    return nullptr;
}
```

## 함수 객체(Functor)

복습을 해보자면 함수 포인터는 이런식으로 사용했었습니다.

```cpp
#include <iostream>

void HelloWorld()
{
    std::cout << "Hello World ! " << std::endl;
}

int main()
{
    void(*pFunc)(void);

    pFunc = &HelloWorld;
    (*pFunc)();

    return 0;
}
```

여기서 `int`를 하나받는 함수를 담을려면 시그니쳐가 다르기 때문에 다른 함수 포인터를 만들어줘야합니다.

```cpp
#include <iostream>

void HelloWorld()
{
    std::cout << "Hello World ! " << std::endl;
}

void HelloNumber(int num)
{
    std::cout << "Hello Number: " << num << std::endl;
}

int main()
{
    void(*pFunc)(void);
    pFunc = &HelloWorld;
    (*pFunc)();

    void(*pFuncInt)(int);
    pFuncInt = &HelloNumber;
    (*pFuncInt)(3);

    return 0;
}
```

우선 함수 포인터는 불편합니다.

함수 객체의 기본은 이렇습니다.

```cpp
class Functor
{
public:
    int operator()()
    {
        return _value;
    }

private:
    int _value = 0;
};

int main()
{
    Functor f;
    std::cout << f() << std::endl;

    return 0;
}
```

`()` 연산자를 오버로딩해주는 겁니다.

그럼 객체가 함수처럼 동작하게 됩니다.

뭐 여기에 C#의 Delegate와 비슷하게 동작하게 하려면 연산자들을 오버로딩해주면 되겠습다.

## 콜백함수

이전에 아이템을 찾는 함수포인터를 건내주는 함수를 만들었던 예시로 돌아가면

```cpp
class Item
{
public:

public:
    int _itemId = 0;
    int _rarity = 0;
    int _ownerId = 0;
};

Item* FindItem(Item items[], int itemCount, bool(*selector)(const Item*))
{
    for (int i = 0; i < itemCount; i++)
    {
        Item* item = &items[i];
        if(selector(item))
            return item;    
    }

    return nullptr;
}
```

이런 식이였어요.

함수포인터 `selector`에 아이템을 찾는 기준이 들어오는 거였습니다. 

하지만 '상태'를 저장할 수 없다는 점이 함수 포인터의 단점이었습니다.

그래서 함수 객체를 이용하면 상태를 저장할 수 있었습니다.

```cpp
class FindByOwnerId
{
public:
    bool operator()(const Item* item)
    {
        return (item->_ownerId == _ownerId);
    }

public:
    int _ownerId;
};

class FindByRarity
{
public:
    bool operator()(const Item* item)
    {
        return (item->_rarity == _rarity);
    }

public:
    int _rarity;
};
```

**Functor** 두개를 만들었습니다.

그럼 저 두가지 Functor를 지원하기 위해서 템플릿을 사용하여 `FindItem`을 수정해줍니다.

```cpp
template<typename T>
Item* FindItem(Item items[], int itemCount, T selector)
{
    for (int i = 0; i < itemCount; i++)
    {
        Item* item = &items[i];
        if(selector(item))
            return item;    
    }

    return nullptr;
}
```

그리고 이렇게 사용하면 되는거죠.

```cpp
int main()
{
    FindByOwnerId functor1;
    functor1._ownerId = 1;
    FindByRarity functor2;
    functor2._rarity = 2;

    Item items[10];
    items[2]._ownerId = 1;
    items[5]._rarity = 2;

    Item* item1 = FindItem(items, 10, functor1);
    Item* item2 = FindItem(items, 10, functor2);

    return 0;
}
```

> # C#

## Delegate

Delegate는 `어떤 메서드를 호출하는 방법`을 담은 객체입니다.

그렇다. 처음 델리게이트를 봤을 때 함수의 시그니처를 담는 느낌이었습니다.

대리자는 C 및 C++의 함수 포인터처럼 메서드를 안전하게 캡슐화하는 형식입니다.

`string`을 인수로 사용하고 `void`를 반환하는 메서드를 캡슐화할 수 있는 `Del` 대리자를 선언합니다.

```csharp
public delegate void Del(string message);
```

대리자 개체는 일반적으로 대리자가 래핑할 **메서드의 이름**을 제공하거나 익명 함수를 사용하여 생성합니다. 대리자를 인스턴스화하고 나면 대리자에 대한 메서드 호출이 대리자에 의해 해당 메서드로 전달됩니다.

```csharp
// Create a method for a delegate.
public static void DelegateMethod(string message)
{
    Console.WriteLine(message);
}
```

```csharp
// Instantiate the delegate.
Del handler = DelegateMethod;

// Call the delegate.
handler("Hello World");
```

대리자 형식은 .NET의 `Delegate` 클래스에서 파생되며, 봉인(`sealed`)되어 있으므로 `Delegate`에서는 파생될 수 없으며 해당 클래스에서 사용자 지정 클래스를 파생할 수도 없습니다.

[Delegate 클래스 (System)](https://docs.microsoft.com/ko-kr/dotnet/api/system.delegate?view=net-5.0)

`Delegate` 클래스

### 멀티캐스팅

`delegate`는 다중캐스트가 가능합니다.

```csharp
Transformer t = Square;
t += AnotherMethod;

t(3); // 호출하면 추가한 순서대로 호출됩니다.

t -= AnotherMethod;
t(3); // 호출하면 Square만 호출됩니다.
```

대리자는 **불변이(immutable) 객체**입니다. 
따라서 += 나 =를 호출하면 실제로는 새로운 대리자 인스턴스가 생성된 후 그것이 기존 변수에 배정됩니다.

### 인스턴스 메서드 vs 정적 메서드

인스턴스의 메서드를 대리자에 등록하는 경우, 해당 인스턴스의에 대한 참조가 필요합니다.

이는 대리자의 `Target` 속성이 인스턴스를 들고 있습니다.

정적 메서드가 등록되어있는 경우 `Target`은 `null`입니다. 

```csharp
public delegate void ProgressReporter (int percentComplete);

class Test
{
		static void Main()
		{
				X x = new X();

				ProgressReporter p = x.InstanceProgress;
				p(99); // 99
				Console.WriteLine(p.Target == x); // True
				Console.WriteLine(p.Method); // Void InstanceProgress(Int32)
		}
}

class X
{
		public void InstanceProgress(int percentComplete)
			=> Console.WriteLine(percentComplete);
}
```

여러 인스턴스의 메서드를 등록한 경우 마지막으로 등록한 인스턴스와 인스턴스의 메서드가 `Target`과 `Method`에 표시됩니다.

- 확인 코드
    
    ```csharp
    using System;
    
    public delegate void ProgressReporter();
    
    class Test
    {
    	static void Main()
    	{
    		X x1 = new X();
    		X x2 = new X();
    		Y y1 = new Y();
    		Y y2 = new Y();
    
    		ProgressReporter p = x1.InstanceProgressX;
    
    		p += x2.InstanceProgressX;
    		p += y1.InstanceProgressY;
    
    		x1.x = 10;
    		x2.x = 20;
    
    		p(); // 10 20 30
    		Console.WriteLine(p.Target == y1); // True
    		Console.WriteLine(p.Target == x2); // False
    		Console.WriteLine(p.Method); // Void InstanceProgressY(Int32)
    
    		p = y2.InstanceProgressY;
    		y2.y = 40;
    		p(); // 40
    
    	}
    }
    
    class X
    {
    	public int x = 30;
    	public void InstanceProgressX()
    		=> Console.WriteLine(x);
    }
    
    class Y
    {
    	public int y = 30;
    	public void InstanceProgressY()
    		=> Console.WriteLine(y);
    }
    ```
    
    ![Untitled](C++%E1%84%8B%E1%85%B4%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%89%E1%85%AE%E1%84%91%E1%85%A9%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%8B%E1%85%AA%20C#%E1%84%8B%E1%85%B4%20Delegate%20417fa929dccb45d9a464a7ae005cad3d/Untitled.png)
    

### 제네릭 대리자 형식

대리자 형식에 제네릭 형식 매개변수를 둘 수도 있다.

```csharp
using System;

public delegate T Transformer<T>(T arg);

public class Util
{
    public static void Transform<T>(T[] values, Transformer<T> t)
    {
        for (int i = 0; i < values.Length; ++i)
            values[i] = t(values[i]);
    }
}

public class Test
{
    static int Square(int x) => x * x;
    public static void Main()
    {
        int[] values = { 1, 2, 3 };
        Util.Transform(values, Square);
        foreach(int i in values)
            Console.WriteLine(i + " ");

    }
}
```

이제 `Transfrom`은 임의의 형식에 대해 작동합니다.

### 표준 Func 대리자와 Action 대리자

제네릭 대리자를 이용하면 임의의 반환 형식과 임의의 개수의 매개변수들을 가진 그 어떤 메서드에도 작동할 정도로 일반적인 대리자 형식들 몇 개만 작성해서 재사용하는 것이 가능해집니다.

`System` 이름공간에 정의된 `Func`와 `Action`이 이러한 예시입니다.

```csharp
delegate TResult Func<out TResult> ();
delegate TResult Func<in T, out TResult> (T arg);
delegate TResult Func<in T1, in T2, out TResult> (T1 arg1, T2 arg2);
//... T16까지 비슷한 선언이 이어짐

delegate void Action();
delegate void Action<in T>(T arg);
delegate void Action<in T1, in T2>(T1 arg1, T2 arg2);
//... T16까지 비슷한 선언이 이어짐
```

실무에서 이 대리자들로 해결되지 않는 유일한 시나리오는 `ref`, `out` 매개변수들과 포인터 매개변수뿐입니다.

### 대리자 대 인터페이스

인터페이스로도 대리자로 해결했던 문제를 해결할 수 있습니다.

```csharp
using System;

public interface ITransformer
{
	int Transform(int x);
}

public class Squarer : ITransformer
{
	public int Transform(int x) => x * x;
}

public class Util
{
	public static void TransformAll(int[] values, ITransformer t)
	{
		for(int i = 0; i < values.Length; ++i)
        {
			values[i] = t.Transform(values[i]);
        }
	}
}

class Test
{
	static void Main()
	{
		int[] values = { 1, 2, 3 };
		Util.TransformAll(values, new Squarer());
		foreach(var i in values)
            Console.WriteLine(i);
	}
}
```

하지만 다음중 하나가 참이라면 인터페이스보단 대리자를 이용하는것이 좋습니다.

- 인터페이스가 메서드를 하나만 정의한다.
- 다중 캐스팅 능력이 필요하다
- 구독자가 인터페이스를 여러 번 구현해야 한다.

우선 위의 예제는 다중 캐스팅은 필요없습니다. 하지만 인터페이스가 메서드 하나만 정의합니다.

게다가 구독자가 여러가지 변환을 지원하려면, 예를 들어 `Square`말고 `Cube`도 지원하려면 `ITransformer`를 여러 번 구현해야 합니다.

여러 번 구현해야 하므로 비슷한 코드를 중복해서 입력해야 하므로 상당히 번거로울 수 있습니다.

# 참고

[Microsoft - 대리자 사용(C# 프로그래밍 가이드)](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/delegates/using-delegates)

[Rookiss - C++ & 언리얼 MMORPG 게임개발 시리즈](https://wondong.notion.site/C-MMORPG-c3f094e6a71b4c41abfb972f5165a338)