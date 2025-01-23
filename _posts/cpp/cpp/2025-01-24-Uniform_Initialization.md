---
title:  "Uniform Initialization" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-01-24
last_modified_at: 2025-01-24
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

---

* 한 번쯤은 아래와 같은 실수를 했봤을 것이다.

```cpp
class A {
 public:
  A() { std::cout << "A 의 생성자 호출!" << std::endl; }
};

int main() {
  A a();
}
```

> 실행 결과
>

* 아무것도 출력 되지 않는다

<br/>

* ```A a();```는 사실 ```A```의 객체 ```a```를 만든것이 아니라 ```A```를 리턴하고 인자를 받지 않는 함수 ```a```를 정의한 것이기 때문이이다. 
* 왜냐하면 C++ 의 컴파일러는 **함수의 정의처럼 보이는 것들은 모두 함수의 정의로 해석** 하기 때문이다.

<br/>

* 심지어 아래와 같은 코드는 더 헷갈린다.

```cpp
class A {
 public:
  A() { std::cout << "A 의 생성자 호출!" << std::endl; }
};

class B {
 public:
  B(A a) { std::cout << "B 의 생성자 호출!" << std::endl; }
};

int main() {
  B b(A());
}
```

* 위 코드도 인자로 ```A```를 리턴하고 인자가 없는 함수를 받으며, 리턴 타입이 ```B```인 함수 ```b```를 정의한 것이다.

<br/>

* 이러한 문제가 발생하는 것은 ```()```가 함수의 인자들을 정의하는데도 사용되고, 그냥 일반적인 객체의 생성자를 호출하는데에도 사용되기 때문이다.
* C++ 11 에서는 이러한 문제를 해결하기 위해 **균일한 초기화(Uniform Initialization)**라는 것을 도입하였다.

<br/>

## Uniform Initialization

* 균일한 초기화는 생성자를 호출하기 위해 ```()```대신 ```{}```를 사용하는 것이다.
* 생성자를 호출하려다 함수를 만들어 버리거나 그런 실수를 방지할 수 있다.

```cpp
class A {
 public:
  A() { std::cout << "A 의 생성자 호출!" << std::endl; }
};

int main() {
  A a{};
}
```

> 실행 결과
> A 의 생성자 호출!

<br/>

* 그런데 ```{}```는 ```()```와 달리 일부 암시적 타입 변환들을 불허한다.

```cpp
class A {
 public:
  A(int x) { std::cout << "A 의 생성자 호출!" << std::endl; }
};

int main() {
  A a(3.5);  // Narrow-conversion 가능
  A b{3.5};  // Narrow-conversion 불가
}
```

* ```A a(3.5);``` 해당 코드는 성공적으로 컴파일 되고 ```x```에는 3.5의 정수 캐스팅 버전이 3이 전달된다.
* 하지만 ```A b{3.5};```는 ```double```인 3.5를 ```int```로 변환할 수 없다는 오류가 발생한다.
* 그 이유는 중괄호를 이용해서 생성자를 호출하는 경우 아래와 같은 암시적 타입 변환들이 불가능하기 때문이다.
  * 따라서 ```{}```를 사용하면 위와 같이 원하지 않는 타입 캐스팅을 방지해서 미연에 오류를 잡아낼 수 있다.

<br/>

* 또 다른 쓰임새로 함수 리턴 시에 굳이 생성하는 객체의 타입을 다시 명시 하지 않아도 된다.

```cpp
class A {
 public:
  A(int x, double y) { std::cout << "A 생성자 호출" << std::endl; }
};

A func() {
  return {1, 2.3};  // A(1, 2.3) 과 동일
}

int main() { func(); }
```

* ```{}```를 이용해서 생성하지 않았더라면 ```A(1, 2.3)```과 같이 클래스를 명시해줘야만 했지만
* ```{}```를 이용할 경우 컴파일러가 알아서 함수의 리턴타입을 보고 추론해준다.

<br/>

## Initializer list(초기화자 리스트)

* 배열을 정의할 때 우리는 다음과 같이 작성했했다.

```int arr[] = {1, 2, 3, 4};```

* 그렇다면 중괄호를 이용해서 마찬가지 효과를 낼 수 없을까? 

```vector<int> v = {1, 2, 3, 4};```

* 놀랍게도 **C++ 11** 에서 부터 이와 같은 문법을 사용할 수 있게 되었었다.

```cpp
class A {
 public:
  A(std::initializer_list<int> l) {
    for (auto itr = l.begin(); itr != l.end(); ++itr) {
      std::cout << *itr << std::endl;
    }
  }
};

int main() { A a = {1, 2, 3, 4, 5}; }
```

> 실행 결과
> 1
> 2
> 3
> 4
> 5

* ```initializer_list```는 우리가 ```{}```를 이용해서 생성자를 호출할 때 
* 클래스의 생성자들 중에 ```initializer_list```를 인자로 받는 생성자가 있다면 전달되어 사용된다.
  * ```()```를 사용해서 호출한다면 ```initializer_list```가 생성되지 않는다.
* C++의 컨테이너를 초기화할 때 간단하게 ```{}```안에 원소들을 컨테이너에 맞게 넣어주면 된다.

<br/>

## initializer_list 사용 시 주의할 점

* 만일 ```{}```를 이용해서 객체를 생성할 경우 생성자 오버로딩 시에 해당 함수가 **최우선**으로 고려된다.
  * 설령 호출시 인자가 알맞지 않더라도 최선을 다해 서 해당 생성자와 매칭한다는 의미이다.

```cpp
class A {
 public:
  A(int x, double y) { std::cout << "일반 생성자! " << std::endl; }

  A(std::initializer_list<int> lst) {
    std::cout << "초기화자 사용 생성자! " << std::endl;
  }
};

int main() {
  A a(3, 1.5);  // Good
  A b{3, 1.5};  // Bad!
}
```

* ```A a(3, 1.5);```는 아무런 문제가 없다.
  * ```()```를 이용해서 생성자를 호출하였기 때문에 A의 첫 번째 생성자인 일반 생성자가 호출!.
* ```A b{3, 1.5};```는 컴파일러가가 initializer_list 를 이용하도록 최대한 노력하려고 하는데
* 1.5 는 ```int```가 아니지만 ```double```에서 ```int```로 암시적 변환을 할 수 있으므로 이를 택하게 됩니다.
  * 문제는 ```{}```는 데이터 손실이 있는 변환을 할 수 없기 때문에 오류가 발생하게 된다.
    * 이러한 문제가 발생하지 않는 경우는 ```initializer_list```의 원소 타입으로 타입 변환 자체가 불가능한 경우여야만 한다다.

```cpp
class A {
 public:
  A(int x, double y) { std::cout << "일반 생성자! " << std::endl; }

  A(std::initializer_list<std::string> lst) {
    std::cout << "초기화자 사용 생성자! " << std::endl;
  }
};

int main() {
  A a(3, 1.5);        // 일반
  A b{3, 1.5};        // 일반
  A c{"abc", "def"};  // 초기화자
}
```

* ```int```나 ```double```이 ```string```으로 변환될 수 없기 때문에 ```initializer_list```를 받는 생성자는 아예 고려 대상에서 제외된다.

<br/>

## initializer_list 와 auto

* 만일 ```{}```를 이용해서 생성할 때 타입으로 ```auto```를 지정한다면 ```initializer_list``` 객체가 생성됩니다\
* ```auto list = {1, 2, 3};```를 하면 ```list```는 ```initializer_list<int>```가 된다다.

<br/>

* 여기서 문제점이 존재한다.

```cpp
auto a = {1};     // std::initializer_list<int>
auto b{1};        // std::initializer_list<int>
auto c = {1, 2};  // std::initializer_list<int>
auto d{1, 2};     // std::initializer_list<int>
```

* 상식적으로 적어도 ```b```는 ```int```로 추론되어야 할 것 같지만, C++ 11 에서는 위 ```a```, ```b```, ```c```, ```d``` 모두 ```std::initializer_list<int>```로 정의된된다.
* 이는 꽤 비상식적이기 때문에 C++ 17 부터 아래와 같이 두 가지 형태로 구분해서 auto 타입이 추론된다.
  * ```auto x = {arg1, arg2...}``` 형태의 경우 ```arg1```, ```arg2``` ... 들이 모두 같은 타입이라면 ```x```는 ```std::initializer_list<T>```로 추론!
  * ```auto x {arg1, arg2, ...}``` 형태의 경우 만일 인자가 단 한 개라면 인자의 타입으로 추론되고, 여러 개일 경우 오류를 발생!

<br/>

* 문자열을 다룰 때 또 한 가지 주의할 점

```auto list = {"a", "b", "cc"};```

* ```list```는 ```initializer_list<std::string>```이 아닌 ```initializer_list<const char*>```이 된다는 점이다. 
* 물론 이 문제는 C++ 14 에서 추가된 리터럴 연산자를 통해 해결 가능하다다.

```cpp
using namespace std::literals;  // 문자열 리터럴 연산자를 사용하기 위해 추가함함
auto list = {"a"s, "b"s, "c"s};
```

* 와 같이 하면 ```initializer_list<std::string>``` 으로 추론된다.

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```