# 스마트 포인터와 상호참조문제

---

C++에서 포인터의 메모리관리를 쉽게 해주기위해 스마트 포인터가 나왔다.

C++에서의 스마트 포인터는 shared_ptr, unique_ptr, weak_ptr, auto_ptr 네가지가 있고 auto_ptr은 C++14에서 삭제되었다.

auto_ptr은 문제가 많아 삭제되었으므로 다루지않고 나머지 세가지를 정리한다.

<br>

> ## RALL(Resource Acquisition Is Initialization) 원칙

먼저 정리하기 전에 스마트 포인터는 RALL 원칙을 따르기위한 객체이다.

- 안전하게 자원을 사용하기 위한, C++에서 자주 쓰는 패턴
- 객체가 사용되는 스코프(범위)를 벗어나면, 자원을 해제해주는 기법
- ex. 함수 내의 지역변수(stack에 할당된 메모리)는 그 함수가 끝나는 시점에서 메모리가 해제되는 원리

<br>

> ## unique_ptr

한 객체의 주소에 대해 **유일 소유권**을 가진다.

대입에 의한 소유권 이전이 금지되어있다.

`move()`를 통해 소유권을 이전하거나 `reset()`으로 소유권을 포기하고 다른 객체를 가리킬수 있다.

- 시범 코드

    ```cpp
    #include <memory>
    #include <iostream>

    int Foo(std::unique_ptr<int>& uPtr)
    {
    	return *uPtr;
    }

    int main()
    {
    	std::unique_ptr<int> uPtrMain(new int(3));
    	std::cout << Foo(uPtrMain) << std::endl;

    	return 0;
    }
    ```

    인자를 reference로 받지않으면 컴파일에러이다.

    왜냐하면 복사가 일어나는데 `unique_ptr`은 유일 소유권이기때문이다. (아마 복사생성자와 대입연산자가 delete되어있을것이다.)

<br>

> ## shared_ptr

- 참조 카운팅 방식 스마트 포인터(Reference Counting Smart Pointer)
- 소유권이 아닌 공유 방식 사용
- 참조 카운트가 0이 될 때만, 해당 객체가 자동으로 삭제됨

shared_ptr에서 상호참조문제가 발생하는데 weak_ptr을 살펴보고 알아보자.

<br>

> ## weak_ptr

- shared_ptr와 함께 사용할 수 있는 스마트 포인터
- shared_ptr의 문제점(상호 참조로 인해 객체가 삭제되지 않는 상황)을 보완하기 위해 사용되는 특수 포인터
- shared*ptr을 weak*ptr로 참조 시, 참조 카운트에 포함되지 않음

<br>

> ## shared_ptr 상호참조문제

아래 코드를 보자.

```cpp
#include <memory>    // for shared_ptr
#include <vector>
 
using namespace std;
 
class User;
typedef shared_ptr<User> UserPtr;
 
class Party
{
public:
    Party() {}
    ~Party() { m_MemberList.clear(); }
 
public:
    void AddMember(const UserPtr& member)
    {
        m_MemberList.push_back(member);
    }
 
private:
    typedef vector<UserPtr> MemberList;
    MemberList m_MemberList;
};
typedef shared_ptr<Party> PartyPtr;
 
class User
{
public:
    void SetParty(const PartyPtr& party)
    {
        m_Party = party;
    }
 
private:
    PartyPtr m_Party;
};
 
 
int _tmain(int argc, _TCHAR* argv[])
{
    PartyPtr party(new Party);
 
    for (int i = 0; i < 5; i++)
    {
        // 이 user는 이 스코프 안에서 소멸되지만,
        // 아래 party->AddMember로 인해 이 스코프가 종료되어도 user의 refCount = 1
        UserPtr user(new User);
 
        // 아래 과정에서 순환 참조가 발생한다.
        party->AddMember(user);
        user->SetParty(party);
    }
 
    // 여기에서 party.reset을 수행해 보지만,
    // 5명의 파티원이 party 객체를 물고 있어 아직 refCount = 5 의 상태
    // 따라서, party가 소멸되지 못하고, party의 vector에 저장된 user 객체들 역시 소멸되지 못한다.
    party.reset();
 
    return 0;
}
```
<br>

`Party`하나와 `User` 다수가 서로를 참조하고 있는 상황이다.

그리고 main에서 마지막 두번째 줄에서 `Party`가 소유권을 포기했다. (`reset()`)

하지만 `Party` 객체는 사라지지않는다. 왜냐하면 다수의 `User`가 참조하고 있기때문에.

그러면 `User` 들도 전부 `reset()`하면 될까? 

아니다. 사라지지않은 `Party` 객체가 `User` 객체들을 `shared_ptr`로 가지고 있어서 메모리 해제가 안된다.

그래서 User가 Party를 `weak_ptr`로 래퍼런싱하고 있는 해결책이 있다.

그렇다면 C#에서 Garbage Collector는 이러한 문제를 어떻게 해결할까.

[Understanding Garbage Collection in .NET](https://www.notion.so/Understanding-Garbage-Collection-in-NET-452d8a2fd0ba4bd6990c5a4d0651d272)