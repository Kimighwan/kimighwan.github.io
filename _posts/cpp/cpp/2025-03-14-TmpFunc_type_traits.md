---
title:  "템플릿 메타 함수와 type_traits" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-03-14
last_modified_at: 2025-03-14
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

## 템플릿 메타 함수

* 함수는 아니지만 마치 함수처럼 작동하는 템플릿 클래스를 템플릿 함수라고 한다.
* 왜 메타 함수인가?, 보통함수는 값에 대해 연산을 하지만 메타 함수는 타입에 대해 연산을 한다.
  * 실제로 함수가 아니라서 ```()```호출 하지 않고 ```<>```에 타입을 전달하여 작업을 한다.

```cpp
template <typename T>
void tell_type() {
  if (std::is_void<T>::value) {
    std::cout << "T 는 void ! \n";
  } else {
    std::cout << "T 는 void 가 아니다. \n";
  }
}

int main() {
  tell_type<int>();

  tell_type<void>();
}
```
> 실행 결과
> T 는 void 가 아니다.
> T 는 void!

<br/>

* 보시다시피 일반적인 함수와 차이점은 템플릿 메타 함수들은 실제론 함수가 아니라는 점
* 타입에 따라서 나누어서 처리가 가능하다.

<br/>

* C++ 표준 라이브러리 중 하나인 ```type_traits``` 에서는 ```is_void```처럼 타입들에 대해서 여러가지 연산을 수행할 수 있는 메타 함수들을 제공하고 있다. 
* 한 가지 더 예를 들어보자면 정수 타입인지 확인해주는 ```is_integral``` 이 있다.

```cpp
class A {};

// 정수 타입만 받는 함수
template <typename T>
void only_integer(const T& t) {
  static_assert(std::is_integral<T>::value);
  std::cout << "T is an integer \n";
}

int main() {
  int n = 3;
  only_integer(n);

  A a;
  only_integer(a);
}
```

* ```static_assert``` 는 C++ 11 에 추가된 키워드로 (함수X.), 인자로 전달된 식이 참인지 아닌지를 컴파일 타임에 확인한다. 
* 다시 말해 ```bool``` 타입의 ```constexpr``` 만 ```static_assert``` 로 확인할 수 있고 그 외의 경우에는 컴파일 오류가 발생...
  * 전달된 식이 참 이라면, 컴파일러에 의해 해당 식은 무시되고, 거짓 이라면 해당 문장에서 컴파일 오류를 발생한다.
* 따라서 ```static_assert``` 와 std::```is_integral``` 을 잘 조합해서 ```T```가 반드시 정수 타입임을 강제할 수 있다.

<br/>

* 이 처럼 메타 함수들을 잘 활용한다면 특정 타입만 받는 함수를 간단하게 작성할 수 있다.

<br/>

## is_class

* ```type_traits```에 정의되어 있는 메타 함수들 중 ```is_class```가 있다. 
* 이 메타 함수는 인자로 전달된 타입이 클래스 인지 아닌지 확인한다
* 아래 처럼 정의가 되어있는데...

```cpp
namespace detail {
template <class T>
char test(int T::*);
struct two {
  char c[2];
};
template <class T>
two test(...);
}  // namespace detail

template <class T>
struct is_class
    : std::integral_constant<bool, sizeof(detail::test<T>(0)) == 1 &&
                                     !std::is_union<T>::value> {};
```

<br/>

* ```std::integral_constant<T, T v>```는 ```v```를 ```static``` 인자로 가지는 클래스이다.
* 쉽게 말해 그냥 어떠한 값을 ```static``` 객체로 가지고 있는 클래스를 만들어주는 템플릿 이라고 생각하면 된다.

<br/>

* 예를 들어서 ```std::integral_constant<bool, false>```는 그냥 ```integral_constant<bool, false>::value```가 ```false```인 클래스이다.

<br/>

* ```sizeof(detail::test<T>(0)) == 1 && !std::is_union<T>::value``` 해당 부분이 ```false```라면 ```is_class```는 그냥 다음과 같고
  
```cpp
template <class T>
struct is_class : std::integral_constant<bool, false> {};
```

* ```is_class::value```는 ```false```가 된다.

> 그렇다면 앞 부분인 ```sizeof(detail::test<T>(0)) == 1``` 은 왜 ```T```가 클래스 일 때만 1 이 될까?

<br/>

## 데이터 멤버를 가리키는포인터 (Pointer to Data member)

* ```int T::*``` 라는 문법은 처음 본다.
* 이는 ```T```의 ```int```멤버를 가리키는 포인터라는 의미다. 아래 예시

```cpp
class A {
 public:
  int n;

  A(int n) : n(n) {}
};

int main() {
  int A::*p_n = &A::n;

  A a(3);
  std::cout << "a.n : " << a.n << std::endl;
  std::cout << "a.*p_n : " << a.*p_n << std::endl;
}
```

* ```p_n```은 `A`의 i```nt```형 멤버를 가리킬 수 있는 포인터를 의미한다.
* 근데 ```p_n```이 실제 존재하는 어떤한 객체의 ```int```형 멤버를 가리키는 것이 아니다.

<br/>

* 그럼 어떻게 사용??
  * ```int A::*p_n = &A::n;``` 처럼 정의했기에 ```p_n```을 역참조해서 해당 객체의 멤버에 접근한다.
    * ```A```의 ```n```을 참조하는 식으로 사용한다.

<br/>

* 그런데 해당 문법은 클래스에만 사용할 수 있다.
* 참고로 해당 클래스에 가리키고자 하는 포인터의 타입인 멤버 변수가 없어도 유효한 문법이다.
  * 다만 아무것도 가리킬 수 없을 뿐...
* 하지만 여기에선 그냥 클래스인지 아닌지 판별하기 위해 사용하는 거라 상관이 없다.

<br/>

```cpp
struct two {
  char c[2];
};
template <class T>
two test(...);
```

* ```test``` 함수의 경우 사실 ```T```가 무엇이냐에 상관없이 항상 인스턴스화 할 수 있다.
* ```detail::test<T>(0)```을 컴파일할 때, 컴파일러는 아래의 둘 중 하나를 선택해야 한다.

```cpp
template <class T>
char test(int T::*);  // (1)

struct two {
  char c[2];
};
template <class T>
two test(...);  // (2)
```

* 그렇다면 ```T```가 클래스라면 1번이 좀더 구체적이므로 (인자가 명시되어 있음) 우선순위가 더 높기 때문에 1번으로 오버로딩 된다. 
* 따라서 ```test<T>(0)```의 리턴 타입은 ```char```이 되고 ```sizeof(char)```은 1이 된다.

<br/>

* 반면 클래스가 아니라면 1번은 불가능한 문법이기에 오버로딩 후보군에서 제외된다.
* 따라서 2번이 유일하게 컴파일되어 리턴타입은 ```two```가 되어 ```sizeof(char c[2])```는 2가 된다.

<br/>

* 뒷쪽에 ```is_union```을 통해 공용체가 아님을 확인하는 이유는
* 데이터 멤버를 가리키는 포인터가 허용되는 것은 **클래스**와 **공용체(union)** 두 가지가 있기 때문이다.

<br/>

---

## 특정 멤버 함수가 존재하는 타입 만을 받는 함수

* 여러가지 메타 함수로 할 수 있었던 것들은 이러이러한 조건을 만족하는 타입을 인자로 받는 함수를 만드는 것이였다.
* 하지만 만약에 이러이러한 멤버 함수가 있는 타입을 인자로 받는 함수를 만들고 싶다면 어떻게 할까?
* 예를 들어서 멤버 함수로 ```func```이라는 것이 있는 클래스만 받고 싶다고 해보자.

```cpp
template <typename T, typename = decltype(std::declval<T>().func())>
void test(const T& t) {
  std::cout << "t.func() : " << t.func() << std::endl;
}

struct A {
  int func() const { return 1; }
};

int main() { test(A{}); }
```
> 잘 작동한다. 만약 ```func```이 정의되어 있지 않은 클래스의 객체를 넘기면 컴파일 오류가 난다.

<br/>

* 만약 리턴 타입까지 정하고 싶다면 ```enable_if```를 사용하면 된다.

```cpp
// T 는 반드시 정수 타입을 리턴하는 멤버 함수 func 을 가지고 있어야 한다.
template <typename T, typename = std::enable_if_t<
                        std::is_integral_v<decltype(std::declval<T>().func())>>>
void test(const T& t) {
  std::cout << "t.func() : " << t.func() << std::endl;
}

struct A {
  int func() const { return 1; }
};

struct B {
  char func() const { return 'a'; }
};

int main() {
  test(A{});
  test(B{});
}
```

<br/>

* 또한 아래와 같이 ```T```가 컨테이너 인지 아닌지 확인하는 방법이 있다
* 컨테이너의 특징은 ```begin```과 ```end```가 정의되어 있다는 사실을 이용하는 것이다.
  * 두 개가 정의되어 있는지 확인하면 된다.

```cpp
template <typename Cont, typename = decltype(std::declval<Cont>().begin()),
          typename = decltype(std::declval<Cont>().end())>
void print(const Cont& container) {
  std::cout << "[ ";
  for (auto it = container.begin(); it != container.end(); ++it) {
    std::cout << *it << " ";
  }
  std::cout << "]\n";
}

int main() {
  std::vector<int> v = {1, 2, 3, 4, 5};
  print(v);

  std::set<char> s = {'a', 'b', 'f', 'i'};
  print(s);
}
```

> ```begin```과 ```end``` 둘 다 정의되어 있지 않은 클래스의 경우 컴파일 오류가 뜬다.

* 한 가지 문제점은 ```typename = ```이 너무 많고 무엇을 하는 것인지 알기가 힘들다
* 또한 가독성이 떨어진다.
* 아래 ```void_t```를 보자

## void_t

* 정의는 아래와 같다.

```cpp
template <class...>
using void_t = void;
```

* 가변길이 템플릿을 이용해서 ```void_t```에 템플릿 인자로 임의의 개수의 타입들을 전달할 수 있고, 어찌 되었든 ```void_t```는 결국 ```void```와 동일하다.
* 만약 올바르지 않은 인자가 있다면 ```void_t```를 사용한 템플릿 함수의 경우 오버로딩 후보군에서 제외가 된다.
* 따라서 아래와 같이 변경이 가능하다

```cpp
template <typename Cont, typename = decltype(std::declval<Cont>().begin()),
          typename = decltype(std::declval<Cont>().end())>
```

```cpp
template <typename Cont,
          typename = std::void_t<decltype(std::declval<Cont>().begin()),
                                 decltype(std::declval<Cont>().end())>>
```

<br/>

* 만약 여기서 사용자가 실수로 템플릿 인자에 컨테이너 말고 인자를 한 개 더 전달한다면 어떻게 될까?
* 오버로딩 후보군에서 제외되지 않아서 그냥 컴파일 오류가 뜬다.

<br/>

* 만약에 위 ```print``` 함수가 표준 라이브러리 함수들 처럼 여러 사용자들을 고려해야 하는 상황이라면
* 위와 같이 사용자가 실수 했을 때에도 정상적으로 작동할 수 있도록 설계해야 한다.
* 이를 위해선 타입 체크하는 부분을 다른 곳으로 빼야 한다.

```cpp
template <typename Cont>
std::void_t<decltype(std::declval<Cont>().begin()),
            decltype(std::declval<Cont>().end())>
print(const Cont& container)
```

> 어렵다 다시 복습하자

```

<br/>

[type_traits](https://en.cppreference.com/w/cpp/header/type_traits)
[static_assert](https://en.cppreference.com/w/cpp/language/static_assert)

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```