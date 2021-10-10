# C# Collections

---

# Array

`Array` 클래스는 모든 배열(1차원과 다차원 모두)의 암묵적인 기반 클래스에며, 표준 컬렉션 인터페이스를 구현하는 가장 근본적인 형식 중 하나이다.

C# 구문으로 선언된 배열에 대해 CLR은 내부적으로 `Array`클래스의 파생 클래스를 만들어 낸다. 

즉, CLR은 배열의 차원들과 원소 형식에 적합한 유사 형식을 합성해 낸다. 

이 유사 형식은 `IList<string>` 같은 형식 있는 제네릭 컬렉션 인터페이스를 구현한다.

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

public class Test
{

    public static void Main()
    {
        int[] intArr = { 1, 2, 3 };
        Console.WriteLine(intArr.GetType() == typeof(Array)); // False
        Console.WriteLine(intArr.GetType()); // System.Int32[]

        Array array = intArr;

        foreach(var i in array)
            Console.WriteLine(i);
    }
}
```

`Array`는 `IList`의 제네릭버전 비제네릭 버전 모두를 구현한다.

단, `IList<T>` 자체는 명시적으로 구현한다. 이는 `Array`의 `public` 인터페이스들에 `Add`나 `Remove` 같은 메서드가 포함되지 않게 하기 위한 것이다.

```csharp
public class MyArray : IList, IList<int>
{
	...

	bool ICollection<int>.Remove(int item)
	{
	    ...
	}
	
	...
}

```

---

# List-like 컬렉션들

이제부터는 List-like 컬렉션들에 초점을 둔다.

그와 다른 Dictionary-like 컬렉션들은 List-like 컬렉션들 후에 설명한다.

이들도 앞에 설명한 인터페이스들처럼 제네릭과 비제네릭버전들이 있다.

비제네릭버전의 인터페이스들은 나름대로 쓸모가 있지만 구체적인 컬렉션들의 비제네릭버전은 하위 호환성을 위해서나 사용된다.

## List<T> 클래스와 ArrayList 클래스

컬렉션 클래스 중 가장 자주 쓰이는 제네릭 `List` 클래스와 비제네릭 `ArrayList` 클래스는 크기를 **동적으로 변경할 수 있는 배열**을 나타낸다.

보통의 배열과 다른 점은 모든 인터페이스를 `public`으로 구현한다는 것과 `Add`와 `Remove` 같은 메서드들도 기대한 대로 작동한다는 것이다.

`List<T>`와 `ArrayList`는 용량을 늘려야 한다면 더 큰 배열로 대체한다. (마치 C++의 std::vector처럼)

요소를 추가하는 것은 효율적이지만, 중간에 요소를 삽입하는 것은 느릴 수 있다.(한칸씩 밀기때문)

정렬 성능은 일반 배열과 비슷하다.

정렬되어 있으면 이진 검색 `BinarySearch` 메서드로 빠른 검색이 가능하다.

[Array.BinarySearch 메서드 (System)](https://docs.microsoft.com/ko-kr/dotnet/api/system.array.binarysearch?view=net-5.0)

[ArrayList.BinarySearch 메서드 (System.Collections)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.arraylist.binarysearch?view=net-5.0)

[List .BinarySearch 메서드 (System.Collections.Generic)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.generic.list-1.binarysearch?view=net-5.0)

`List<T>`와 `ArrayList`는 기존 컬렉션들을 받는 생성자를 제공하는데, 이는 기존 컬렉션의 요소들을 새 `List<T>`나 `ArrayList`로 복사한다.

`ArrayList`는 .NET Framework 1.x 코드와의 하위 호환성을 위해 쓰인다.

지저분한 캐스팅때문에 이제는 잘 쓰이지않는다.

```csharp
ArrayList al = new ArrayList();
al.Add("Hello");
string first = (string)al[0];
string[] strArr = (string[])al.ToArray(typeof(string));
```

컴파일러가 컴파일타임에 이를 검증하지 못하여 다음은 실행중에 오류가 난다.

```csharp
int first = (int)al[0];
```

`System.Linq`에 있는 `Cast`메서드와 `ToList`메서드를 차례로 호출하면 `ArrayList`를 `List`로 변환할 수 있다.

```csharp
ArrayList al = new ArrayList();
al.AddRange(new[] { 1, 5, 9 });
List<int> list = al.Cast<int>().ToList();
```

`Cast`메서드와 `ToList`메서드는 `System.Linq.Enumerable`에 있는 확장 메서드이다.

## LinkedList<T> 클래스

`LinkedList<T>`는 이중 연결 목록 자료주고를 나타내는 제네릭 클래스이다. (C++의 std::list처럼)

빠르게 중간 삽입이 가능하다는 특징이 있다.

노드를 삽입할 위치를 찾는 것이 느릴 수 있어서 위의 장점이 퇴색될 수는 있다.

인덱스로 특정 노드를 접근하려면 순회해야해서 효율적인 이진 검색인 `BinarySearch` 메서드를 제공하지 않는다.

[LinkedList 클래스 (System.Collections.Generic)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.generic.linkedlist-1?view=net-5.0)

## Queue<T> 클래스와 Queue 클래스

`Queue<T>`와 `Queue`는 선입선출 FIFO를 특징으로 갖는 자료구조이다.

열거가능 Enumerable 컬렉션이지만 `IList<T>`와 `IList`는 구현하지 않아 인덱스 임의 접근은 불가능하다. 임의 접근이 필요하다면 `ToArray`로 요소들을 배열에 복사하면 된다.

[Queue 클래스 (System.Collections.Generic)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.generic.queue-1?view=net-5.0)

내부적으로 List와 같이 배열로 원소들을 관리하므로 크기가 부족하면 더 큰 배열로 대체한다.

## Stack<T> 클래스와 Stack 클래스

`Stack<T>`와 `Stack`은 후입선출 LIFO를 특징으로 갖는 자료구조이다.

이 자료구조도 `ToArray`를 제공하고 해당 자료구조 자체로는 임의접근을 할 수 없다.

[Stack 클래스 (System.Collections.Generic)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.generic.stack-1?view=net-5.0)

## BitArray 클래스

`BitArray`는 `bool`값들을 빽빽하게 저장하는 컬렉션으로 각 원소가 1바이트만 저장하여 `List`보다 메모를 효율적으로 사용한다.

그리고 비트별 연산자 메서드 `And`, `Or`, `Xor`, `Not`을 제공한다.

```csharp
var bits = new BitArray(2);
bits[1] = true;

bits.Xor(bits);
Console.WriteLine(bits[1]); // False
```

[BitArray 클래스 (System.Collections)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.bitarray?view=net-5.0)

## HashSet<T> 클래스와 SortedSet<T> 클래스

`HashSet<T>`와 `SortedSet<T>`는 각각 .NET Framework 3.5와 4.0에 추가된 제네릭 컬렉션이다.

- `Contains`메서드는 해시 기반 조회 hash-based loopup 를 이용해서 빠르게 실행된다.
- 중복된 원소를 저장하지 않으며, 중복된 원소를 추가하는 요청은 소리없이 무시한다.
- 인덱스로 특정 원소에 접근할 수 없다.

> HashSet은 C++의 std::unordered_set / SortedSet은 C++의 std::set으로 생각해도 좋을듯
> 

`HashSet<T>`는 키들만 저장하는 **해시테이블**로 구현되어있고, `SortedSet<T>`는 **적흑 트리 red-black tree**로 구현되어있다.

`SortedSet<T>`는 원소들을 정렬된 순서로 저장하는 반면 `HashSet<T>`는 그렇지 않다.

조금 특이한 메서드로는 집합연산을 수행하는 메서드들이다.

```csharp
public void UnionWith(IEnumerable<T> other); // 합집합, 원소 추가
public void IntersectWith(IEnumerable<T> other); // 교집합, 원소 삭제
public void ExceptWith(IEnumerable<T> other); // 차집합, 원소 삭제
public void SymmetricExceptWith(IEnumerable<T> other); // 대칭차집합, 원소 삭제
```

기존 집합을 수정한다는 의미에서 파괴적 destructive 메서드이다.

`UnionWith`는 둘째 집합의 모든 원소들을 첫 집합에 추가한다.

`IntersectWith`는 둘 중 한 집합에만 있는 원소들을 제거한다.

`ExceptWith`는 둘째 집합의 원소들을 첫 집합에서 제거한다.

`SymmetricExceptWith`는 두 집합에 모두 있는 원소들을 첫 집합에서 제거한다.

반면 다음 메서드들을 그냥 집합에 대한 질의를 수행하므로 파괴적이지 않은 집합 연산 메서드이다.

```csharp
public bool IsSubsetOf(IEnumerable<T> other);
public bool IsProperSubsetOf(IEnumerable<T> other);
public bool IsSupersetOf(IEnumerable<T> other);
public bool IsProperSupersetOf(IEnumerable<T> other);
public bool Overlaps(IEnumerable<T> other);
public bool SetEquals(IEnumerable<T> other);
```

매개변수로 `IEnumerable<T>`를 받는 것은 `HashSet<T>`과 `SortedSet<T>`뿐만 아니라 다른 컬렉션도 받을 수 있다는 뜻이다.

[HashSet 클래스 (System.Collections.Generic)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.generic.hashset-1?view=net-5.0)

[SortedSet 클래스 (System.Collections.Generic)](https://docs.microsoft.com/ko-kr/dotnet/api/system.collections.generic.sortedset-1?view=net-5.0)

---

# Dictionary-like

이제부터 Dictionary-like 컬렉션들을 설명한다.

사전 Dictionary는 각 요소가 키-값 쌍인 컬렉션이다.

.NET Framework는 사전을 위한 표준 프로토콜을 정의하는데, `IDictionary`와 `IDictionary<TKey, TValue>`이다.

그리고 범용 사전 클래스들로도 구성된다.

범용 사전 클래스들은 다음과 같은 시준으로 분류될 수 있다.

- 항목들을 정렬된 순서로 저장하는가
- 키뿐만 아니라 위치로도 항목들에 접근 가능한가
- 제네릭인지 아니면 비제네릭인지 여부
- 큰 사전에서 키로 항목을 추출하는 것이 빠른지 아니면 느린지 여부

[사전 클래스들](https://www.notion.so/3b15ab75f43f4092a9fd79405b82d816)

키로 조회하는 시간 복잡도는 다음과 같다.

- `Hashtable`과 `Dictionary`, `OrderedDictionary`는 $O(1)$
- `SortedDictionary`와 `SortedList`는 $O(log n)$
- `ListDictionary` 등 다른 `List<T>`같은 비사전 형식들을 $O(n)$

## IDictionary<TKey, TValue> 인터페이스

`IDictionary<TKey, TValue>`는 모든 키-값 쌍 기반 컬렉션의 표준 프로토콜을 정의한다.

`ICollection<T>`를 확장하여 구현한다.

```csharp
public interface IDictionary<TKey, TValue> : 
	ICollection<KeyValuePair<TKey, TValue>>, IEnumerable
{
	bool ContainsKey(TKey key);
	bool TryGetValue(TKey key, out TValue value);
	void Add(TKey key, TValue value);
	bool Remove(TKey key);

	TValue this [TKey key] { get; set; }
	ICollection<TKey> Keys { get; }
	ICollection<TValue> Values { get; }
}
```

사전에 어떤 항목을 추가할 때에는 `Add`메서드를 호출할 수도 있고 인덱서의 설정접근자를 사용할 수도 있다.  

후자는 주어진 키가 존재하면 갱신하고 아니면 추가한다.

전자는 주어진 키가 존재하면 예외가 발생한다.

항목을 조회하는 방법도 두가지이다.

인덱서를 사용하거나 `TryGetValue`메서드를 호출할 수 있다.

`TryGetValue`메서드는 해당 항목이 존재하지 않으면 `false`를 반환한다.

인덱서를 사용했는데 해당 항목이 존재하지 않으면 예외가 발생한다.

`IDictionary<TKey, TValue>`를 열거하면 다음과 같은 `KeyValuePair`구조체들의 순차열을 얻게 된다.

```csharp
public struct KeyValuePair<TKey, TValue>
{
	public TKey Key { get; }
	public TValue Value { get; }
}
```

## IDictionary 인터페이스

비제네릭 `IDictionary` 인터페이스는 기본적으로 다음과 같이 제네릭 인터페이스와의 차이점이 있다.

- 존재하지 않는 키를 조회하려는 경우 인덱서는 `null`을 돌려준다.
- 키의 존재 여부를 판정하는 메서드가 `ContainsKey`가 아니라 `Contains`이다.
- 열거를 수행하면 `DictionaryEntry`구조체들의 순차열이 반환된다.
    
    ```csharp
    public struct DictionaryEntry
    {
    	public object Key { get; set; }
    	public object Value { get; set; }
    }
    ```
    

## Dictionary<TKey, TValue> 클래스와 Hashtable 클래스

제네릭 `Dictionary<TKey, TValue>`는 해시 테이블 자료구조에 키들과 값들을 저장하므로 빠르고 효율적이다.

이 클래스의 바탕 해시테이블은 이런 식으로 작동한다.

요소를 추가할 때 해시테이블은 주어진 요소의 키를 정수 해시코드(일종의 유사 고유 값)으로 변환하고, 그 해시코드를 적당한 알고리즘을 이용해서 해시 키로 변환한다. 그리고 내부적으로 이 해시 키를 이용해서 주어진 요소의 값을 집어넣을 통 Bucket을 결정한다.

만일 통에 여러 개의 값이 들어 있으면 선형 검색을 이용해서 해당 값을 찾는다. 좋은 해시 함수는 진정으로 고유한 해시코드를 계산하는 데 주력하는 대신, 32비트 정수 공간에 고르게 분호되는 해시코드를 생성하는 데 주력한다. 그러면 몇몇 통에 값들이 몰리는 현상을 피할 수 있다.

`Dictionary`의 키 형식으로는 상등 비교가 가능하고 해시 코드를 얻을 수 있는 형식이기만 하면 어떤 형식도 가능하다.

상등 비교는 `object.Equal`메서드가 사용되며, 유사고유 해시코드를 얻는 데에는 `GetHashCode`메서드가 쓰인다. 그 이외에는 두 메서드를 재정의하거나 사전을 생성할 때 적절한 `IEqualityComparer`객체를 제공하면 된다.

```csharp
var d = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase);
```

이에 관해서는 곧 '상등 및 순서 비교 플러그인'에서 다룬다.

이 클래스의 비제네릭 버전은 `Hashtable`이다. `IDictionary`의 비제네릭 버전만 구현한다는 차이점이 있다.

그리고 `Dictionary`와 `Hashtable` 둘다 항목들이 정렬되지 않는다.

## OrderedDictionary 클래스

`OrderedDictionary`는 비제네릭 사전 클래스로 요소들의 저장 순서가 추가 순서와 동일하다는 특징이 있다. 이 덕분에 키뿐만 아니라 인덱스로도 접근이 가능하다.

이름으로 오해를 할 수 있는데 `OrderedDictionary`는 항목들이 정렬되어 있지 않다.

이 클래스의 제네릭 버전은 없다.

## ListDictionary 클래스와 HybridDictionary 클래스

`ListDictionary`는 단일하게 연결된 목록 singly linked list, 줄여서 단일 연결 목록 자료구조에 바탕 자료를 저장한다. 

이 클래스는 정렬 기능을 제공하지 않지만, 애초에 항목들이 추가된 순서는 유지한다.

목록이 클 수록 `ListDictionary`는 극도로 느려진다.

유일한 장점은 작은 목록에서 효율적이라는 점...

`HybridDictionary`는 `ListDictionary`와 `Hashtable`의 조합이다.

이 사전은 처음에는 `ListDictionary`이지만, 일정한 크기가 되면 `Hashtable`로 전환한다. (목록 크기에 따른 `ListDictionary`의 성능 문제를 피하기 위해)

하지만 대체로 `Dictionary`의 메모리 사용량이나 성능이 그리 나쁘지는 않기 때문에 그냥 `Dictionary`를 사용해도 무방하다...

두 클래스 모두 비제네릭 버전만 있다.

## 정렬된 사전

요소들을 항상 키를 기준으로 정렬된 상태로 유지하는 사전 클래스가 두 개 있다,

- `SortedDictionary<TKey, TValue>`
- `SortedList<TKey, TValue>`

`SortedDictionary<,>`는 **적흑 트리 red-black tree**를 사용한다. (마치 C++의 std::map처럼)

적흑 트리는 모든 삽입 또는 조회 상황에서 일관된 성능을 내도록 설계된 자료구조이다.

`SortedList<,>`는 내부적으로 배열 두 개에 키들과 값들을 담는다.

정렬된 배열에 대해 이진 검색을 적용하므로 조회가 빠르지만, 새 항목을 추가할 때마다 기존 항목들을 옮겨서 자리를 만들어야 하기 때문에 삽입 성능은 나쁘다.

무작위한 순차열 삽입은 `SortedDictionary<,>`가 더 빠르다.

그러나 `SortedList<,>`는 키뿐만 아니라 인덱스로도 접근이 가능한 추가 능력을 갖추고 있다.

`SortedDictionary<,>`는 n번째 항목을 접근하려면 차례로 순회해야한다.