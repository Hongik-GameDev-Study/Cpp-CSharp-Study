# Unity Coroutines in Detail

Unity Coroutine은 어떻게 작동하는걸까?

엔진 내부는 볼 수 없으니 C#의 기능을 통해 추측해보고자 한다.

우선 Unity Technology에서 Unity Engine내부를 어떻게 구현했는지는 모른다.

하지만 짐작은 할 수 있다.

두가지 단서로 말이다.

```csharp
IEnumerator UnityCoroutine()
{
	// ...
	yield return null;
	// ...
}
```

- `IEnumerator`타입을 리턴해야 함.
- `yield return` 키워드를 사용한다.

이 둘은 C#의 문법이며 Unity가 만든게 아니다. 그러므로 Unity는 C#의 문법을 이용하여 Coroutine 기능을 제공한다고 추측해볼 수 있다.

## What is Iterators ?

Iterator 반복자는 Collection의 종류와 상관없이 Collection의 원소들을 순회하는 기능을 제공한다.

이는 `IEnumerator`와 `IEnumerable`을 통해 캡슐화되어있다.

[IEnumerator Interface (System.Collections)](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerator?view=net-5.0)

`IEnumerator`

[IEnumerable Interface (System.Collections)](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerable?view=net-5.0)

`IEnumerable`

보면 `IEnumerator`는 `MoveNext`, `Current`와 같은 순회하는 기능을 가지고 있고, `IEnumerable`은 `IEnumerator`를 반환하는 `GetEnumerator`를 가지고 있다.

그리고 순회하는 기능은 `foreach`문과도 연동되어있어 모든 Collection은 `foreach`문을 사용할 수 있는 것이다.

이는 조금 독특한데 직접 구현하여 foreach문을 돌려보면 알 수 있다.

```csharp
public static void Main()
{
    MyCollection myCollection = new MyCollection ();
    foreach (var e in myCollection)
    {
				//...
    }

}

public class MyCollection : IEnumerable<int>
{
    private int[] data = { 1, 2, 3 };
    public IEnumerator GetEnumerator()
    {
        return data.GetEnumerator();
    }

    IEnumerator<int> IEnumerable<int>.GetEnumerator()
    {
        foreach (var e in data)
            yield return e;
    }
}
```

대충이나마 구현해보았다.

보면 `IEnumerator`는 구현하지 않았는데, 이는 `yield return`의 마법에 숨겨져있다.

컴파일러가 `yield return`을 만나면 이 함수는 반복자임을 알아차리고 `IEnumerator`를 구현해준다.

직접 구현해서 보고자하는 부분은 `yield return`이다.

`yield return`을 하여 함수를 빠져나오고 다음 실행시 `yield return`의 다음 문장부터 실행한다.

이는 Unity의 Coroutine과 매우 흡사한 실행 방식이며 Unity는 이를 역이용하여 Coroutine을 구현한 것으로 보인다.

컴파일러가 직접 구현해주지 않게 하려면 우리가 직접 구현하면 된다.

```csharp
public class MyIntList : IEnumerable
{
    int[] data = { 1, 2, 3 };

    public IEnumerator GetEnumerator()
    {
        return new Enumerator(this);
    }

    class Enumerator : IEnumerator
    {
        MyIntList collection;

        int currentIndex = -1;

        public Enumerator(MyIntList collection)
        {
            this.collection = collection;
        }

        public object Current
        {
            get
            {
                if (currentIndex == -1)
                    throw new InvalidOperationException("열거가 시작되지 않았음.");
                if (currentIndex == collection.data.Length)
                    throw new InvalidOperationException("목록의 끝을 지나쳤음.");
                return collection.data[currentIndex];
            }
        }

        public bool MoveNext()
        {
            if (currentIndex >= collection.data.Length - 1) return false;
            return ++currentIndex < collection.data.Length;
        }

        public void Reset()
        {
            currentIndex = -1;
        }
    }

}
```

## Using IEnumerator as Coroutine

위에서 말했듯이 함수의 중간에 `yield return`하여 빠져나오고 다음 호출에서 `yield return` 다음 문장부터 다시 시작한다는 점이 눈여겨볼만한 점이다.

하지만 컬렉션에서 이를 구현하는 것과 별개로 인터페이스는 구현측에서 다르게 구현하는 것도 가능하기에 Unity에서는 이를 역이용한다.

유니티가 아닌 C#에서 이를 작성해보자.

```csharp
using System;
using System.Collections;
using System.Threading;

public class Test
{
    private static bool playingAnimation = true;

    private static bool playingCinematic = true;

    public static IEnumerator CoroutineFunction()
    {
        Console.WriteLine("Start Of Coroutine");

        Console.WriteLine("Playing Animation");

        while (playingAnimation)
            yield return null;

        Console.WriteLine("Playing Cinema");

        while (playingCinematic)
            yield return null;

        Console.WriteLine("End Of Coroutine");
    }

    public static void BoolChanger()
    {
        Thread.Sleep(3000);
        playingAnimation = false;
        Thread.Sleep(3000);
        playingCinematic = false;
    }

    public static void Main()
    {
        Thread t1 = new Thread(BoolChanger);
        t1.Start();

        IEnumerator e = CoroutineFunction();
        while (e.MoveNext()) { }
    }
}
```

위의 코드에서 

```csharp
IEnumerator e = CoroutineFunction();
while (e.MoveNext()) { }
```

이 부분이 유니티의 `StartCoroutine`으로 생각하면 된다.

실행해보면 코루틴과 비슷하게 실행되는 것을 볼 수 있다.

`CoroutineFunction`은 Unity의 Coroutine과 똑같이 생각하면 될 것이다. 

각 `bool`변수들이 `false`가 될 때까지 프레임을 계속 쉬어준다.

그리고 `Main`에선 `BoolChanger`를 다른 스레드에서 실행시켜준다.

3초를 쉬어주고 각 `bool`변수들을 `false`로 만들어준다.

`yield return`으로 무언가를 반환하면 반환한 값은 `IEnumerator.Current`에 대입된다.

`yield return`을 만났을 때 `IEnumerator`를 만들어주는 컴파일러덕분에 가능하다.

```csharp
while (e.MoveNext()) { }
```

이 부분에서 `IEnumerator`의 `MoveNext`를 계속 호출하는데 `MoveNext`의 처음부터가 아니라 마지막에 실행을 멈춘 `yield return`부터 시작을 하는 것을 알 수 있다.

## Unity3D Coroutines in Nutshell

이걸 유니티에서 해볼까?

총 두개의 스크립트를 만들 것이고 하나는 Coroutine을 실행하는 `CoroutineManager` 스크립트이고 하나는 Coroutine을 등록하는 `CoroutineRegisterer` 스크립트이다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoroutineRegisterer : MonoBehaviour
{
    [SerializeField] public CoroutineManager coroutineManager;

    [SerializeField] private bool playingAnimation = true;

    [SerializeField] private bool playingCinematic = true;

    // Start is called before the first frame update
    void Start()
    {
        coroutineManager.StartCoroutine(CoroutineFunction());
    }

    public IEnumerator CoroutineFunction()
    {
        Debug.Log("Start Of Coroutine");

        Debug.Log("Playing Animation");

        while (playingAnimation)
            yield return null;

        Debug.Log("Playing Cinema");

        while (playingCinematic)
            yield return null;

        Debug.Log("End Of Coroutine");
    }
}
```

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoroutineManager : MonoBehaviour
{
    List<IEnumerator> unblockedCoroutines = new List<IEnumerator>();

    // Update is called once per frame
    void Update()
    {
        List<IEnumerator> shouldRunNextFrame = new List<IEnumerator>();

        foreach (IEnumerator coroutine in unblockedCoroutines)
        {
            if (!coroutine.MoveNext()) // Coroutine is finished
                continue;

            if(!(coroutine.Current is YieldInstruction)) // null or other type than YieldInstruction
            {
                shouldRunNextFrame.Add(coroutine);
                continue;
            }

            unblockedCoroutines = shouldRunNextFrame;
        }

    }

    public new void StartCoroutine(IEnumerator coroutine)
    {
        unblockedCoroutines.Add(coroutine);
    }
}
```

인스펙터에서 `bool`변수들을 차례로 `false`로 만들어주면 Coroutine이 `while`문을 빠져나와 다음 `yield return`으로 갈 수 있는 것을 볼 수 있다.

아마도 `CoroutineManager`같은 클래스를 `Monobehaviour`가 상속받아 `StartCoroutine`이나 `StopCoroutine`을 그대로 상속받은 것이 아닌가 추측해볼 수 있다.

그래서 면접에서 받았던 질문인

> "유니티에서 코루틴은 마치 멀티스레드처럼 동작하는 것처럼 동작하는데, 유니티는 싱글스레드입니다. 이는 어떻게 한 것일까요?"

에 대한 답을 할 수 있게 되었다.

---

## 참고

[Unity Coroutines: How Do They Work?](https://gamedevunboxed.com/unity-coroutines-how-do-they-work/)

[Unity3D coroutines in detail](https://blog.csdn.net/StupidCodeGenerator/article/details/11526285)
