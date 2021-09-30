# STL, vector의 push_back과 emplace_back

```csharp
template <class T>
void std::vector<T>.push_back(T instance);

template <class T, Args...> // 가변 인자 템플릿
void std::vector<T>.emplace_back(T's Args...)
```

`push_back`은 인자로 객체 자체를 넣는다.

```csharp
class Item
{
public:
    int healAmount;
    Item(int amount) : healAmount(amount) { cout << "일반 생성자 호출" << endl; };
    Item(const Item& rhs) : healAmount(rhs.healAmount) { cout << "복사 생성자 호출" << endl; };
    Item(Item&& rhs) : healAmount(move(rhs.healAmount)) { cout << "이동 생성자 호출" << endl; };
    ~Item() { cout << "소멸자 호출" << endl; };
};

int main()
{
    vector<Item> items;
    items.push_back(Item(1000)); 

    // 1. Item(1000)을 통해 임시 객체 생성
    // 2. 복사 생성자를 통해 push_back 함수에서 또 다른 임시 객체 생성
    // 3. vector의 끝에 요소 추가
    // 4. main의 임시 객체 삭제
    // 5. vector 내 임시 객체 삭제
}
```

![https://postfiles.pstatic.net/MjAyMTA5MjVfNTcg/MDAxNjMyNTA2Nzk0OTY1.PJTEl2O9uW4CzNOV3VFABLlfWeZKio7qFsXuuRfNOikg.RSE-wmC7oU9G5zI-wWwxHDKO4LitnII-vvCHcuHbuEAg.PNG.psh50zmfhtm/image.png?type=w966](https://postfiles.pstatic.net/MjAyMTA5MjVfNTcg/MDAxNjMyNTA2Nzk0OTY1.PJTEl2O9uW4CzNOV3VFABLlfWeZKio7qFsXuuRfNOikg.RSE-wmC7oU9G5zI-wWwxHDKO4LitnII-vvCHcuHbuEAg.PNG.psh50zmfhtm/image.png?type=w966)

요런 식으로 총 2번의 생성 및 삭제 과정을 거치게 된다. 그럼 `emplace_back()` 은?

---

```csharp

```

![https://postfiles.pstatic.net/MjAyMTA5MjVfMTcx/MDAxNjMyNTA2Nzc1ODkw.-Y6m_86rFPGl7aSJT_xnvpOvAk0nJNCPTS-TraXD8OQg.9CC_2Xd2XaceTMfA3m21dZ-OooZv222Ix258FuU3buIg.PNG.psh50zmfhtm/image.png?type=w966](https://postfiles.pstatic.net/MjAyMTA5MjVfMTcx/MDAxNjMyNTA2Nzc1ODkw.-Y6m_86rFPGl7aSJT_xnvpOvAk0nJNCPTS-TraXD8OQg.9CC_2Xd2XaceTMfA3m21dZ-OooZv222Ix258FuU3buIg.PNG.psh50zmfhtm/image.png?type=w966)

불필요한 임시 객체가 생성되지 않게 되어 총 1번의 생성 및 삭제 과정만 거치게 된다.

---

`emplace_back`은 `push_back`으로 할 수 있는 모든 것을 할 수 있으며, 위와 같이 매우 효율적이다.

// 그럼 push_back을 우리가 주로 사용하는 이유가 뭘까?

그 원인은 `push_back`과 `emplace_back`의 생성자 차용 방식에 있다.

`push_back`은 명시적인 생성자를 호출하게 되어 상관없지만,
`emplace_back`은 가변 매개변수 리스트를 받기 때문에 컴파일 타임엔 타당한 생성자가 존재하는 지 알 수 없다.

다음을 보자.

![https://postfiles.pstatic.net/MjAyMTA5MjVfMTE5/MDAxNjMyNTA3OTM5NDky.s26iKDanzqcNHBjIjK7UhhDMd4rye4Qpj23_Qd6ujXwg.8hpIlgUS10KvRUiu2R3Oc0bb1bHHEVv4-Kt9DytsZ94g.PNG.psh50zmfhtm/image.png?type=w966](https://postfiles.pstatic.net/MjAyMTA5MjVfMTE5/MDAxNjMyNTA3OTM5NDky.s26iKDanzqcNHBjIjK7UhhDMd4rye4Qpj23_Qd6ujXwg.8hpIlgUS10KvRUiu2R3Oc0bb1bHHEVv4-Kt9DytsZ94g.PNG.psh50zmfhtm/image.png?type=w966)

![https://postfiles.pstatic.net/MjAyMTA5MjVfNDUg/MDAxNjMyNTA3OTY0Nzg3.oE01zr9nmMbG6ECxMUVbXEq_5zLmvI57N1fs1EF1OxIg.aICJ6QL6BSxCDN3Z_rcsF043n4fE0pz2Bwch46fmtzwg.PNG.psh50zmfhtm/image.png?type=w966](https://postfiles.pstatic.net/MjAyMTA5MjVfNDUg/MDAxNjMyNTA3OTY0Nzg3.oE01zr9nmMbG6ECxMUVbXEq_5zLmvI57N1fs1EF1OxIg.aICJ6QL6BSxCDN3Z_rcsF043n4fE0pz2Bwch46fmtzwg.PNG.psh50zmfhtm/image.png?type=w966)

내 작성 의도는 2차원 벡터 `vec`의 첫 번째 요소에 10을 넣고자 하는 것이었다.

`push_back`의 경우, 명시적으로 `vector<T>` 타입의 인자를 요구하기 때문에 

> 'No known conversion for argument 1 from ‘int’ to ‘const value_type& {aka const std::vector<int>&}’

![https://postfiles.pstatic.net/MjAyMTA5MjVfMTc4/MDAxNjMyNTA4NTQ1ODMx.0W44upxHksvZi7HyY-iuozrFM8dDRqQUUUI07xa3MRgg.f5dQpU33NzIQ0ULxaxcT_7siUDCK4hN9yMYAjQFwgU8g.PNG.psh50zmfhtm/image.png?type=w966](https://postfiles.pstatic.net/MjAyMTA5MjVfMTc4/MDAxNjMyNTA4NTQ1ODMx.0W44upxHksvZi7HyY-iuozrFM8dDRqQUUUI07xa3MRgg.f5dQpU33NzIQ0ULxaxcT_7siUDCK4hN9yMYAjQFwgU8g.PNG.psh50zmfhtm/image.png?type=w966)

내부적으로 `vector<vector<int>> vec(10);` 를 불렀음

---

이와 같이 `emplace_back`은 이동 간의 비용을 절약해주지만 컴파일 타임에 모든 문제를 잡아낼 수 없다는 리스크가 존재한다.

아래처럼 이동 작업의 cost가 방대한 경우가 아니라면 굳이 사용할 필요 없을 것 같다.
실제로 컴파일러의 최적화에 따라 `push_back`과 속도도 크게 차이나지 않는다고 한다.

```csharp
class Image {
  Image(size_t w, size_t h);
  // 엄청나게 큰 녀석~
};

std::vector<Image> images;
// images.push_back(Image(2000, 1000)); // 엄청나게 큰 녀석을 임시 객체에 담고, 옮기고, 삭제해야함
images.emplace_back(2000, 1000); // 이렇게 한 번에!
```

![https://postfiles.pstatic.net/MjAyMTA5MjVfNTMg/MDAxNjMyNTA4NDYxOTE0.awgWw7tAwPgcE0IxAgNpzVgME6_G1EbNGsv-6U35flwg.Me4NumBq_7Hgx0ff4rgS_qN2xhLiTBm0FLIxMqaUlnUg.PNG.psh50zmfhtm/image.png?type=w966](https://postfiles.pstatic.net/MjAyMTA5MjVfNTMg/MDAxNjMyNTA4NDYxOTE0.awgWw7tAwPgcE0IxAgNpzVgME6_G1EbNGsv-6U35flwg.Me4NumBq_7Hgx0ff4rgS_qN2xhLiTBm0FLIxMqaUlnUg.PNG.psh50zmfhtm/image.png?type=w966)

문자열을 넣어도 빨간 줄이 안 뜬다

---

참고 자료

[https://gumeo.github.io/post/emplace-back/](https://gumeo.github.io/post/emplace-back/)
