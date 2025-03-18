---
title:  "SFINAE & enable_if" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-03-18
last_modified_at: 2025-03-18
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

## Substitution failure is not an error - SFINAE

- 컴파일 오류 시 오버로딩 후보군에서 제외된다??

```cpp
template <typename T>
void test(typename T::x a) {
  std::cout << "T::x \n";
}

template <typename T>
void test(typename T::y b) {
  std::cout << "T::y \n";
}

struct A {
  using x = int;
};

struct B {
  using y = int;
};

int main() {
  test<A>(33);

  test<B>(22);
}
```

- ```test<A>(33);```의 경우 템플릿 인자로 ```A```를 전달했으니 각 템플릿함수는 아래와 같이 치환된다.

```cpp
void test(A::x a) { std::cout << "T::x \n"; }   // 1
void test(A::y b) { std::cout << "T::y \n"; }   // 2
```

- 여기서 ```A```에는 ```y```가 존재하지 않기 때문에 두 번째 문법은 틀린 문법이다
- 하지만 컴파일 오류가 발생하지 않는다
- **SFINAE** 원칙에 의해 템플릿 인자 치환 후에 만들어진 식이 문법적으로 틀려도 컴파일 오류를 발생 시키는 대신 함수의 오버로딩 후보군에서 제외만 시킨다.

<br/>

- 그런데 컴파일러가 템플릿 인자 치환 시에 함수 내용 전체가 문법적으로 올바른지 확인하지 않는다.
- 단순히 함수의 인자들과 리턴 타입만이 문법적으로 올바른지만 확인한다.
- 따라서, 함수 내부에서 문법적으로 올바르지 않은 내용이 있더라도 오버로딩 후보군으로 남아있는다.

<br/>

- 이런식으로 원하는 타입에 대해서만 작동하는 함수를 만들 수 있다.
- ```type_traits```에는 해당 작업을 쉽게하게 도와주는 메타 함수가 있는데, ```enable_if```이다.

## enable_if

- 이 함수는 **SFINAE**를 통해서 조건에 맞지 않는 함수들을 오버로딩 후보군에서 쉽게 뺄 수 있게 도와주는 템플릿 메타 함수이다.
- 아래 처럼 정의되어 있다.

```cpp
template<bool B, class T = void>
struct enable_if {};

template<class T>
struct enable_if<true, T> { typedef T type; };
```

- ```B```부분에 확인하고 싶은 조건을 전달하면 된다.
  - 참이라면 ```enable_if::type```의 타입이 ```T```가 되고
  - 거거짓이라면 ```enable_if```에 ```type```가 존재하지 않게 된다.

<br/>

- 아래와 같이 정수 타입인지 확인할 수 있다.

```cpp
template <typename T,
          typename = typename std::enable_if<std::is_integral<T>::value>::type>
void test(const T& t) {
  std::cout << "t : " << t << std::endl;
}

struct A {};

int main() { test(A{}); }
```

- 만약 위에서 정수 타입이 아닌 객체를 전달하면 오버로딩이 가능한 것이 없다고 컴파일 오류가 뜬다.
- 그리고 ```typename =``` 부분은 템플릿에 디폴트 인자를 전달하는 부분인데, 원래에는 ```typename U =```처럼 템플릿 인자를 받지만 이 경우 저 식 자체만 필요하기 때문에 굳이 인자를 정의할 필요가 없다.
- 그리고 ```std::enable_if``` 앞에 추가적으로 ```typename```이 또 붙는 이유는 ```std::enable_if<>::type```이 의존 타입 이기 때문.
  

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```