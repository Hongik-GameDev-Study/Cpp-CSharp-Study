# Unity Coroutines in Detail

---

엔진 내부는 볼 수 없으니 C#의 기능을 통해 추측해보고자 한다.

아마도 C#의 Iterator (반복자: `IEnumerator`)를 이용하여 구현했을 것으로 보인다.

그래서 Coroutine을 구현할 때 `IEnumerator`를 반환하는 함수를 정의하는 것이다.

## Understanding IEnumerator

`IEnumerator`와 `IEnumerable`은 C#의 모든 컬렉션이 상속받아 구현하는 인터페이스이다.

이 인터페이스들은 컬렉션이 데이터를 어떻게 저장하는지를 숨긴 채 순회하는 기능을 제공한다.

[IEnumerator Interface (System.Collections)](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerator?view=net-5.0)

`IEnumerator`

[IEnumerable Interface (System.Collections)](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerable?view=net-5.0)

`IEnumerable`

이를 왜 설명하냐면 Unity Coroutine은 C#의 `IEnumerator`를 사용한 일종의 트릭이기때문이다.

```csharp
IEnumerator LongCompution()
{
	while(someCondition)
	{
		yield return null;
	}
}
```

이런식으로 Unity Coroutine을 사용할 수 있다.

하지만 이게 C#과 무슨 상관일까?

우선 Unity Technology에서 Unity Engine내부를 어떻게 구현했는지는 모른다.

하지만 짐작은 할 수 있다.

두가지 단서로 말이다.

- `IEnumerator`타입을 리턴해야 함.
- `yield return` 키워드를 사용한다.

`IEnumerator`는 C#의 컬렉션에서 사용된다.

이는 컬렉션의 요소를 하나씩 반환하기위해 사용하는데, 요소가 무한개여도 프로그램이 막히지않는다.

`IEnumerator`에서 정의하는 `Current` 프로퍼티와 `MoveNext` 메서드로 이를 제공한다.

함수의 중간에 `yield return`하여 빠져나오고 다음 `MoveNext`에서 `yield return`부터 다시 시작한다는 점이 눈여겨볼만한 점이다.

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

`yield return`으로 무언가를 반환하면 반환한 값은 `IEnumerator.Current`에 대입된다.

하지만 `null`을 반환하여 하나의 프레임을 쉬는 것보다는 다른 것을 하고 싶을 수 있다.

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

아마도 `CoroutineManager`같은 클래스를 `Monobehaviour`가 상속받아 `StartCoroutine`이나 `StopCoroutine`을 구현한 것이 아닐까 싶다.

그래서 면접에서 받았던 질문인

> "유니티에서 코루틴은 마치 멀티스레드처럼 동작하는 것처럼 동작하는데, 유니티는 싱글스레드입니다. 이는 어떻게 한 것일까요?"

에 대한 답을 할 수 있게 되었다.

---

## 참고

[Unity Coroutines: How Do They Work?](https://gamedevunboxed.com/unity-coroutines-how-do-they-work/)

[](https://blog.csdn.net/StupidCodeGenerator/article/details/11526285)