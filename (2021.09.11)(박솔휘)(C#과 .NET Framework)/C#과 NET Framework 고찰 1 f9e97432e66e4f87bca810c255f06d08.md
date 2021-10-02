# C#과 .NET Framework 고찰 1

C# 언어는 플랫폼 중립적이며, .NET Framework와 함께 잘 작동되도록 만들어졌다.

## **1. C#의 특징**

**통합 Type 체계**

- 모든 Type(내장 Type이든, 아니든)은 하나의 공통 기능 형식을 공유한다.
>> 어떤 Type의 인스턴스든 ToString()으로 문자열 변환이 가능하다.

```cpp
[클래스 이외의 여러 사용자 정의 Type]
 
- 인터페이스 등
// C#은 JAVA와 마찬가지로 다중 상속을 지원하지 않으나, 인터페이스는 다중 상속이 가능하다.
```

```cpp
[Method 이외의 함수]

- Property, Event 등의 함수

// C#은 객체 지향 언어이나 함수형 프로그래밍에서도 영향을 받았다.

// 1. Delegate의 존재, 함수를 값으로서 전달할 수 있다.
// 2. 람다식과 질의 표현식의 존재
```

```cpp
[강력한 Static Type 언어]

- 컴파일 시점에 Type Safety를 강력히 규제한다.
// while(1) 등 묵시적 변환도 불가능
```

```cpp
[동적 메모리 관리]

- C#은 CLR에 의존하여 메모리를 자동 관리한다.

// C++ 동적 할당에서의 해제 문제가 해결되어, 포인터 사용 문제가 사라진다.
// 포인터는 Unsafe 표시 블록에서만 사용이 가능해졌다.
```

```cpp
[멀티 플랫폼 지원]

- 본디 C#은 .NET을 이용한 Windows 플랫폼에서 실행되는 코드를 작성하는데에만 사용되었으나, 
  현재는 다양한 프레임워크의 지원으로 크로스플랫폼이 가능하다.
```

## **2. .NET Framework**

- .NET Framework는 Windows 플랫폼 어플리케이션 개발을 위한 라이브러리의 집합이다.
// .NET Framework는 CLR와 BCL(Base Class Language)로 구성된다.
- C# 뿐만 아니라 F#, VB(Visual Basic), .NET, Managed C++ 등의 언어를 지원한다.
// 위 언어는 .NET에 의한 Managed Language라고 할 수 있다.
- C# (Managed Langauge)는 언어 종속적인 C# 컴파일러에 의해 Managed Code로 컴파일된다.
- 이 Managed Code는 "IL"이라는 중간 언어로 표현되는데, JIT( in CLR )는 이 IL을 기계어 코드로 변경하는데 사용된다. // CLR은 IL to Machine Code 기능의 JIT 외에도 메모리 관리 등 많은 역할을 가진다.
- IL은 CLR을 거쳐 컴퓨터 고유의 .exe로 변경되고, .dll과 .meta 파일들과 함께 Assembly라는 단위로 묶여 패키징된다.

[CLI] // CLI(Common Language Infrastructure)

- CLI는 여러 고급 언어가 다른 컴퓨터 플랫폼에서 특정한 구조를 위해 별도의 수정 과정 없이 사용될 수 있게 만들어 주는 오픈 규격이다.
// CLI은 "규격"이며, "기능"이 아님

1. CTS // Common Type System

- .NET Framework에서 각 Type의 메모리 표현 방식을 정하는 표준 제공 // Common Language 간의 정보 교환 위함

2. CLS // Common Language Specification 

- .NET Framework를 지원하는 Managed Language의 언어 스펙 표준 제공 // Common Language 간의 기능 지원 통일 위함

3. VES // Virtual Execute System

- Common Language의 가상 실행 환경 제공

4. Metadata

[CLR] // CLR은 .NET Framework식 CLI 구현 형태를 말한다.

* 주로 알려진 .NET의 CLR 내 기능들은 다음과 같다.

1. JIT 컴파일러 // Just In Time Comfiler

- IL ㅡ> OS Native Code 기능 제공 // 여러 고급 언어의 기계어화 과정을 통합할 수 있게 됨

// JIT 방식은 실행 시에만 얻을 수 있는 프로그램 분석 정보를 사용하여 높은 성능을 달성할 수 있다. 
// 특히 객체 지향 언어나 스크립트 언어 같이 동적 언어인 경우 효과적이라고 한다.
// 하지만 실행 시 프로그램 분석과 컴파일을 함께 수행하는데 추가 메모리 및 CPU 사이클이 필요한 단점이 있다.

2. AOT 컴파일러 // Ahead Of Time Comfiler

- 런타임에 IL을 Native Code으로 변환하는 과정이 부담스럽기 때문에, 런타임 전에 Native Code로 변환해주는 기능 제공

// 컴파일한 코드의 크기가 크고 실행 시간 정보를 이용한 최적화가 불가능해 
// 오래 실행되는 서버 프로그램 등에서 기대할 수 있는 추가적인 성능 향상을 꾀할 수는 없다.
// 주로 내장 임베디드 시스템에 사용되는 편

[BCL] // Base Class Language

- CIL(Common Intermediate Language) 언어로 되어 있는 표준 라이브러리 

// BCL은 명명할 때 계층 구조를 구분하기 위해 점(.)으로 명명한다. 
// 예를 들면 수학적 계산을 하기 위해 만들어진 이름 공간인 Math 이름 공간은 System.Math로 명명된다.

**3. WinRT?**

C#과 .NET을 작성하며 딸려오는 프레임워크가 있길래 추가 작성해본다.

WinRT는 Windows RunTime 라이브러리의 준자이며, 
Windows Store 어플리케이션을 작성하기 위해(모바일 환경에 적응하기 위해) 제작된
Windows 8 이상에서 제공되는 언어 독립적 Microsoft COM 방식 API 집합이다.

// 내부는 C++로 작성되어있어 C#, VB 등 뿐 아니라 C++, JS 등에서도 사용할 수 있다.

WinRT는 현대식 인터페이스, 모바일 기기 고유 기능 등을 지원하기 위한 기능에 더해 .NET과 유사한 서비스도 제공한다.

Visual Studio에는 이렇게 .NET과 겹치는 부분을 감추어 주는 Windows Store 프로젝트용 Reference Profile이 존재한다.
// .NET 참조 어셈블리들의 집합이라고도 함

Reference Profile은 .NET Framework 중 필요하지 않은 부분을 모두 감추며,
MS Store(Windows Store)는 이런 감추어진 영역으로의 접근을 모두 거부해준다.
// Sandboxing (보안 강화), 규모 축소

- 하지만 이런 WinRT가 .NET Framework를 모두 대체해준다고 착각하면 안된다.
// 표준 데스크톱, 서버 개발 등에는 .NET Framework가 권장된다.
1. 작성한 프로그램과 다른 프로그램과의 통신이 극도로 제한된다.
2. Windows Store에 극도로 의존하게 된다.
3. 프로그램 작동이 Windows8 이상으로 제한된다.

자세히는 모르므로 여기까지만 작성해야겠다. // 책 더 읽어봐야지

****
