# Move semantics and rvalue references in C++11

---

```cpp
#include <iostream>
 
using namespace std;
 
**vector<int>** doubleValues (const vector<int>& v)
{
    vector<int> new_values;
    new_values.reserve(v.size());
    for (auto itr = v.begin(), end_itr = v.end(); itr != end_itr; ++itr )
    {
        new_values.push_back( 2 * *itr );
    }
    **return new_values;**
}
 
int main()
{
    vector<int> v;
    for ( int i = 0; i < 100; i++ )
    {
        v.push_back( i );
    }
    **v = doubleValues( v );**
}
```

위의 코드를 보면 **`return new_values;`**에서 복사가 한번 일어난다. (new_value → temp)

그리고 **`v = doubleValues( v );`**에서 ****두번째 복사가 일어난다. (temp → v)

하지만 이는 불필요한 복사이다. 우리의 목적은 new_value → v 일뿐이기때문...

그저 temp를 v로 **이동(move)**시켜주는 문법은 없을까.

그래서 C++11에서 이를 해결해주는 ravlue reference와 move semantics가 등장한다.

먼저 rvalue부터 살펴본다.

<br>

<hr>

## Detecting temporary objects with rvalue references

```cpp
const string& name = getName(); // ok
string& name = getName(); // NOT ok
```

밑에 것이 안되는 이유는 곧 사라질 temporary object를 mutable reference로 binding하기때문이다. 곧 사라질 객체의 값을 바꿀수 있으면 위험할 것이다.

그래서 mutable reference가 아닌 const reference로 해야한다.

그리고 주소를 봐도 다른 객체(temp와 지역변수 `name`)임을 알 수 있다

- 시범 코드

    ```cpp
    #include <string>
    std::string nameWondong = "wondong";

    std::string getName()
    {
    	return nameWondong;
    }

    int main()
    {
    	const std::string& name = getName(); // ok
    	//std::string& name = getName(); // NOT ok

    	return 0;
    }
    ```

그래서 C++11부터는 새로운 종류의 reference가 나왔다. 이름은 **rvalue reference**.

rvalue reference는 mutable reference로 rvalue를 binding할 수 있게 해준다.

```cpp
const string&& name = getName(); // ok
string&& name = getName(); // also ok - praise be!
```

하지만 이게 뭐.

rvalue reference는 함수의 매개변수로 선언할 때 그 진가를 발휘함.

```cpp
printReference (const String& str)
{
        cout << str;
}
 
printReference (String&& str)
{
        cout << str;
}
```

이제 함수의 인자로 lvalue를 넣으면 위것이 호출되고, rvalue를 넣으면 밑에것이 호출된다.

```cpp
string me( "alex" );
// calls the first printReference function, taking an lvalue reference
printReference(  me ); 
// calls the second printReference function, taking a mutable rvalue reference
printReference( getName() ); 
```

- 시범 코드

    ```cpp
    #include <string>
    #include <iostream>
    std::string nameWondong = "wondong";

    std::string getName()
    {
    	return nameWondong;
    }

    void printReference(const std::string& str)
    {
    	std::cout << str;
    }

    void printReference(std::string&& str)
    {
    	std::cout << str;
    }

    int main()
    {
    	std::string me("alex");
    	// calls the first printReference function, taking an lvalue reference
    	printReference(me);
    	// calls the second printReference function, taking a mutable rvalue reference
    	printReference(getName());

    	return 0;
    }
    ```

    밑은 클래스로 시험한 것. 소멸자를 만들어줘야 구분이 쉽다.

    ```cpp
    #include <iostream>

    class RRefTestObject
    {
    public:
    	virtual ~RRefTestObject()
    	{
    		std::cout << "~RRefTestObject()" << std::endl;

    		if(ip != nullptr)
    			delete ip;
    	}

    	RRefTestObject(int x)
    		: i(x), ip(new int(x))
    	{ 
    		std::cout << "RRefTestObject()" << std::endl; 
    	}

    	RRefTestObject(const RRefTestObject& rhs) 
    	{ 
    		i = rhs.i; 
    		ip = new int(*rhs.ip);
    		std::cout << "RRefTestObject(const RRefTestObject& rhs)" << std::endl; 
    	}

    	RRefTestObject& operator=(const RRefTestObject& rhs)
    	{
    		i = rhs.i;
    		ip = new int(*rhs.ip);
    		std::cout << "operator=(const RRefTestObject& rhs)" << std::endl;
    		return *this;
    	}

    	RRefTestObject(RRefTestObject&& rhs) 
    	{ 
    		i = rhs.i; 
    		ip = rhs.ip;
    		rhs.ip = nullptr;

    		std::cout << "RRefTestObject(RRefTestObject&& rhs)" << std::endl; 
    	}

    	RRefTestObject& operator=(RRefTestObject&& rhs) noexcept
    	{
    		i = rhs.i;
    		ip = rhs.ip;
    		rhs.ip = nullptr;

    		std::cout << "operator=(RRefTestObject&& rhs)" << std::endl;
    		return *this;
    	}

    private:
    	int i;

    	int* ip;
    };

    RRefTestObject global(10000);

    RRefTestObject getGlobal()
    {
    	return global;
    }

    RRefTestObject getTemp()
    {
    	return RRefTestObject(1);
    }

    void printReference(const RRefTestObject& lref)
    {
    	std::cout << "printReference(const RRefTestObject& lref)" << std::endl;
    }

    void printReference(RRefTestObject&& rref)
    {
    	std::cout << "printReference(RRefTestObject&& rref)" << std::endl;
    }

    int main()
    {
    	RRefTestObject mainObject(1234);
    	// calls the first printReference function, taking an lvalue reference
    	printReference(mainObject);
    	// calls the second printReference function, taking a mutable rvalue reference
    	printReference(getGlobal());

    	return 0;
    }
    ```

이게 왜 진가냐하면 복사의 횟수가 줄어들기때문.

<br>

<hr>

## Move constructor and move assignment operator

사용자정의 타입(class)의 rvalue reference를 다룰 때 move constructor와 move assignment oprator를 사용해야 한다.

move constructor는 copy constructor와 비슷한데, 같은 타입의 object를 받아 새로운 object instance를 리턴한다.

하지만 copy constructor와 달리 memory allocation을 피해 object의 field들을 copy하지않고 **move**하게 정의한다.

field를 move한다고..?

이뜻은 primitive type(int, float 등등)는 그냥 copy하고 포인터는 그냥 deep copy하지않고  pointer에 들어있는 주소값만 가져온다. 그리고 원래 pointer는 null로 초기화한다. (마치 훔친것 같은)

move constructor로 들어온 매개변수 object는 temporary object이기때문에 이래도 된다.

마치 다음과 같이

```cpp
class ArrayWrapper
{
public:
    // default constructor produces a moderately sized array
    ArrayWrapper ()
        : _p_vals( new int[ 64 ] )
        , _size( 64 )
    {}
 
    ArrayWrapper (int n)
        : _p_vals( new int[ n ] )
        , _size( n )
    {}
 
    // move constructor
    **ArrayWrapper (ArrayWrapper&& other)
        : _p_vals( other._p_vals  )
        , _size( other._size )
    {
        other._p_vals = NULL;
        other._size = 0;
    }**
 
    // copy constructor
    ArrayWrapper (const ArrayWrapper& other)
        : _p_vals( new int[ other._size  ] )
        , _size( other._size )
    {
        for ( int i = 0; i < _size; ++i )
        {
            _p_vals[ i ] = other._p_vals[ i ];
        }
    }
    ~ArrayWrapper ()
    {
        delete [] _p_vals;
    }
 
private:
    int *_p_vals;
    int _size;
};
```

집중해야할 점은

1. The parameter is a non-const rvalue reference
2. other._p_vals is set to NULL

1번째가 아니고 const 가 들어오면 **`other._p_vals = NULL;`** 가 불가능해진다. 

하지만 왜 NULL로 초기화해야할까?

그건 temporary object가 스코프를 벗어나 소멸자가 호출되면 `_p_vals`를 해제하기 때문이다. 

그럼 move로 `_p_vals`값을 얻은 객체의 `_p_vals`가 해제되는 꼴이다.

그리고 반환타입을 `const`로 해버리면 move constructor가 아닌 copy constructor가 호출되므로 다음과 같이하면 좋지않다.

```cpp
// makes the move constructor useless, the temporary is const!
const ArrayWrapper getArrayWrapper (); 
```

그리고 field의 타입이 사용자정의 class타입일때가 있다.

예를 들어 다음 클래스가 있다고 하자.

```cpp
class MetaData
{
public:
    MetaData (int size, const std::string& name)
        : _name( name )
        , _size( size )
    {}
 
    // copy constructor
    MetaData (const MetaData& other)
        : _name( other._name )
        , _size( other._size )
    {}
 
    // move constructor
    MetaData (MetaData&& other)
        : _name( other._name )
        , _size( other._size )
    {}
 
    std::string getName () const { return _name; }
    int getSize () const { return _size; }
    private:
    std::string _name;
    int _size;
};
```

단순히 `string _name`과 `int _size`를 들고 있는 클래스이다. 이동생성자와 복사생성자도 별것없다.

이제 `MetaData`를 포함하는 `ArrayWrapper`클래스를 보자.

```cpp
class ArrayWrapper
{
public:
    // default constructor produces a moderately sized array
    ArrayWrapper ()
        : _p_vals( new int[ 64 ] )
        , _metadata( 64, "ArrayWrapper" )
    {}
 
    ArrayWrapper (int n)
        : _p_vals( new int[ n ] )
        , _metadata( n, "ArrayWrapper" )
    {}
 
    // move constructor
    **ArrayWrapper (ArrayWrapper&& other)
        : _p_vals( other._p_vals  )
        , _metadata( other._metadata )
    {
        other._p_vals = NULL;
    }**
 
    // copy constructor
    ArrayWrapper (const ArrayWrapper& other)
        : _p_vals( new int[ other._metadata.getSize() ] )
        , _metadata( other._metadata )
    {
        for ( int i = 0; i < _metadata.getSize(); ++i )
        {
            _p_vals[ i ] = other._p_vals[ i ];
        }
    }
    ~ArrayWrapper ()
    {
        delete [] _p_vals;
    }
private:
    int *_p_vals;
    **MetaData _metadata;**
};
```

틀린 곳을 찾으시오.

**`_metadata( other._metadata )`** 이 부분이 틀렸다.

왜냐하면 other는 rvalue reference이지만 lvalue이다. 

그러므로 **`_metadata( other._metadata )`** 는 move constructor가 아닌 copy constructor를 호출함.

temporary object는 원래 사라지기 직전의 object이지만,

move constructor에 들어간 temporary object는 다시 새로운 스코프에서 새로운 생명을 얻는다.

그럼 우선 우리가 가진건 lvalue(`other`)이고 필요한건 rvalue이다.

lvalue를 rvalue로 바꿀수 있다면 참 좋겠다.

<br>

<hr>


## std::move

그래서 있다.

```cpp
// move constructor
ArrayWrapper (ArrayWrapper&& other)
    : _p_vals( other._p_vals  )
    , _metadata( **std::move(other._metadata)** )
{
    other._p_vals = NULL;
}
```

그리고 std::string도 그냥 복사를 했던 MetaData 클래스도 바꿔줘야겠다.

```cpp
// move constructor
MetaData (MetaData&& other)
    : _name( **std::move(other._name)** )
    , _size( other._size )
{}
```

<br>

<hr>


## Returning an explicit rvalue-reference from a function

```cpp
#include <iostream>

int x = 2;

int getInt()
{
    return x;
}

int&& getRvalueInt()
{
    // notice that it's fine to move a primitive type--remember, std::move is just a cast
    return std::move(x);
}

void printAddress(const int& v) // const ref to allow binding to rvalues
{
    std::cout << reinterpret_cast<const void*>(&v) << std::endl;
}

int main()
{
    printAddress(getInt());
    printAddress(x);

    printAddress(getRvalueInt());
    printAddress(x);

	return 0;
}
```

![Untitled](Move%20semantics%20and%20rvalue%20references%20in%20C++11%209e161d92b4e043908f4adf63238b5330/Untitled.png)

## 참고자료

[Rvalue References and Move Semantics in C++11 - Cprogramming.com](https://www.cprogramming.com/c++11/rvalue-references-and-move-semantics-in-c++11.html)