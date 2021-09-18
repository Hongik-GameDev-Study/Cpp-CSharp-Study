# C++ Casting Operators

## static_cast

<br>


이것이 C++의 일반적인 변환 연산자.

<br>


> Otherwise, an expression `e` can be explicitly converted to a type `T` using a `static_cast` of the form `static_cast<T>(e)`, if the declaration `T t(e);` is well-formed,

<br>

한 객체 타입과 다른 객체 타입이 연관이 있다면 변환에 성공한다.

위에서 설명하는 것은 표현식 `e`가 `T`타입으로 `static_cast`으로 변환할 수 있는 것은 `T t(e);` 가 가능해야 한다.

<br>

```cpp
int i(2.4f);

float f = 2.4f;
float* fp = &f;
void* vp(fp);
```

<br>

그래서 연관이 있다는 것은 `float` → `int` 같은 연관도 있고 `pointer` → `void*` 같은 연관도 있다고 할 수 있다.

<br>

```cpp
float f = 2.4f;
int i = static_cast<int>(f);

float* fp = &f;
void* vp = static_cast<void*>(fp);
```

<br>


> ### **변환 생성자**

<br>


내장 타입에서 사용자 정의 타입으로의 변환을 앞으로 설명한다.

변환 생성자는 인자를 하나받는 모든 생성자를 변환생성자라고 부른다.

<br>


```cpp
class Score
{
public:
	Score(int score) : _score(score) {}

	int Get() { return _score; }

private:
	int _score;
};
```

<br>


`Score`라는 클래스가 있고 `int`를 하나 받는 변환 생성자가 있다.

이게 왜 변환 생성자냐면 이제 `int`를 `Score`로 암시적으로 변환하는 것이 가능해졌기 때문.

<br>


```cpp
class Game
{
public:
	Game() : _score(0) {}

	void SetScore(Score newScore)
	{
		_score = newScore.Get();
	}

private:
	int _score;
};

int main()
{
	Score score(4);
	Game game;
	game.SetScore(score);
	Score secondScore = 2;

	return 0;
}
```

<br>


`Score`를 받는 메서드의 인자로 `int`를 넣으면 암시적 변환이 일어나 `Score`로 변환된다.

그리고 초기화나 대입도 암시적 변환이 일어난다.

<br>


```cpp
Score secondScore = 2;
```

<br>


하지만 이는 좀 부자연스럽고 예상치못할 수 있다.

그래서 보통 `explicit`키워드를 붙여 이러한 변환은 미연에 막는다.

<br>


```cpp
explicit Score(int score) : _score(score) {}
```

<br>


그리고 이로써 static_cast로 명시적 변환을 해줘야하게 된 것이다.

<br>


```cpp
Score secondScore = static_cast<Score>(2);
```
<br>

---

## dynamic_cast

<br>

> `dynamic_cast` can only be used with pointers and references to classes (or with `void*`). Its purpose is to ensure that the result of the type conversion points to a valid complete object of the destination pointer type.
>
> This naturally includes pointer **upcast** (converting from pointer-to-derived to pointer-to-base), in the same way as allowed as an implicit conversion.
>
> `dynamic_cast` can also **downcast** (convert from pointer-to-base to pointer-to-derived) polymorphic classes (those with virtual members) if -and only if- the pointed object is a valid complete object of the target type.

<br>

대충 위는 `dynamic_cast`는 클래스의 포인터나 참조자에만 사용가능하고, 클래스를 Polymorphic class으로 만들어야 `dynamic_cast`가 작동한다라는 말이다.

<br>


> A class that declares or inherits a virtual function is called a polymorphic class.

<br>


Polymorphic class란, `virtual` 함수를 선언했거나 상속받은 클래스이다.

<br>


```cpp
class Score
{
public:
	Score(int score) : _score(score) {}
	virtual ~Score() {}

	int Get() { return _score; }

protected:
	int _score;
};

class ScoreDerived1 : public Score
{
public:
	ScoreDerived1() : Score(0) {}
};

class ScoreDerived2 : public Score
{
public:
	ScoreDerived2() : Score(0) {}
};

int main()
{
	Score* score1 = new ScoreDerived1();
	Score* score2 = new ScoreDerived2();

	ScoreDerived1* scored1 = dynamic_cast<ScoreDerived1*>(score1);

	return 0;
}
```

<br>

그래서 보통 소멸자를 `virtual`로 선언하면 `dynamic_cast`는 작동한다.

하지만 `virtual` 함수가 하나도 없으면 작동하지 않는다는 것을 알 수 있다

![Untitled](C++%20Casting%20Operators%204ef9d74ac0d842598a07ec9bea0bb4cd/Untitled.png)

<br>

---

## const_cast

`const_cast`를 설명하기 전에 `const`의 두가지를 살펴보자.

<br>


>  ### **상위 const vs 하위 const**

<br>

포인터는 포인터가 `const`인지 가리키는 객체가 `const`인데 개별적으로 이야기가 가능하다.

그에 따라 상위 `const`와 하위 `const`로 나뉘는데, 해당 변수의 값을 바꿀 수 없는게 상위 `const`이고, 가리키는 대상을 바꿀 수 없는게 하위 `const`이다.

<br>


```cpp
int i = 0;
int* const p1 = &i; // p1 값을 바꿀 수 없음. 상위 const
const int ci = 42; // ci는 바꿀 수 없음. 상위 const
const int* p2 = &ci; // p2는 바꿀 수 있음. 하위 const
const int* const p3 = p2; // 상위와 하위 const는 서로 독립적으로 사용 가능
const int &r = ci; // 참조자 타입에 있는 const는 항상 하위
```

<br>


>  ### **const_cast가 바꿀 수 있는 것**

<br>

`const_cast`는 하위 `const`만을 바꾼다.

즉 포인터나 참조자가 가리키는 객체의 `const` 유무를 바꿀 수 있다는 뜻이다.

<br>

```cpp
const char* pc;
char* p = const_cast<char*>(pc);
```

<br>

하지만 `const`인 객체를 `const_cast`로 바꿀 수 있게 하는 겻의 결과는 미정의이다.

[Is this undefined behavior with const_cast?](https://stackoverflow.com/questions/25209838/is-this-undefined-behavior-with-const-cast)



```cpp
const int e = 3; // const인 객체
int *ptr_e = const_cast<int *>(&e); // const가 아닌 포인터로 참조
std::cout << &e << ' ' << ptr_e << '\n'; // 우선 주소는 같게 나옴

*ptr_e = 5; // 하지만 값을 바꿔주면
std::cout << *ptr_e << '\n'; // ?!
std::cout << e << '\n'; // ?!
```

<br>


![Untitled](C++%20Casting%20Operators%204ef9d74ac0d842598a07ec9bea0bb4cd/Untitled%201.png)

<br>


결국 `const_cast`를 사용할 때는 `const`로 선언하지 않은 변수를 가리키는 pointer 나 reference에 대해, 포인터 간 `const_cast`를 사용하여 `const` 한 속성을 없앤다.

<br>

---

## reinterpret_cast

<br>


포인터나 참조자에 할 수 있는 캐스팅.

역참조한 객체의 비트 해석을 재해석한다.

<br>

```cpp
#include <iostream>

int main()
{
	int i = 3;
	int* rip = &i;
	float* rfp = reinterpret_cast<float*>(rip);

	std::cout << *rip << " " << *rfp << std::endl;

	int* sip = &i;
	float sfp = static_cast<float>(*sip);

	std::cout << *sip << " " << sfp << std::endl;

	return 0;
}
```

![Untitled](C++%20Casting%20Operators%204ef9d74ac0d842598a07ec9bea0bb4cd/Untitled%202.png)

<br>

`static_cast`와 비교할만한데, `static_cast`는 객체를 변환하면서 계산하여 비트를 바꾼다.

두번째 줄에 `int`인 `3`을 `float`인 `3.0f`으로 바꿀려면 비트가 바뀌어야 한다는 것은 알고 있을 것이다.

하지만 첫번째 출력은 `int`인 `3`을 그대로 `float`으로 해석하여 저런 결과가 나온 것.

<br>

---

## 구식 캐스팅

<br>

포함한 타입에 따라 구식 캐스트는 `const_cast`, `static_cast`, `reinterpret_cast`와 같은 동작을 함.

결국 구식 캐스팅으로 위에서 살펴본 모든 것이 가능하지만, 그 부분을 나눠서 안전성을 도모한 것.

<br>

```cpp
type(expr); // 함수 형식 캐스트 표기법
(type)expr; // C 언어 형식 캐스트 표기법
```