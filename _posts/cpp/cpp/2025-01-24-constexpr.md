---
title:  "constexpr" 

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

## constexpr

* ```constexpr``` 키워드는 객체나 함수 앞에 붙일 수 있는 키워드로
* **해당 객체나 함수의 리턴 값을 컴파일 타임에 알 수 있다** 라는 의미를 전달하는 것이다.
  
* 컴파일 타임에 어떠한 식의 값을 결정할 수 있다면 해당 식을 **상수식 (Constant expression)** 이라고 표현한다.
* 상수식들 중에서 값이 정수인 것을 정수 ** 정수 상수식 (Integral constant expression)** 이라고 한다.

<br/>

* 객체의 정의에 ```constexpr```가 오게 된다면 해당 객체는 어떠한 상수식에도 사용될 수 있다.
* 언뜻 보기에 ```const```와 큰 차이가 없어 보인다.

## constexpr VS const

* 같은 상수로 값을 변경할 수 없다는 공통점이 있다.
* 하지만 ```const```는 굳이 컴파일 타임에 그 값을 알 필요가 없다.

```cpp
int a;
const int b = a;
constexpr int c = a;
```

* b는 컴파일 타임에 값을 알 수 없지만 값을 지정해주면 값을 바꿀 수 없다는 것은 확실하다.
* 반면에 ```constexpr``` 변수의 경우 반드시 오른쪽에 다른 상수식이 와야 한다.
  * 하지만 컴파일 타임 시에 ```a```가 뭐가 올 지 알 수 없다. 
    * 따라서 위 코드는 컴파일 오류가 된다.
* ```constexpr```은 항상 ```const```이지만 ```const```는 ```constexpr```이 아니다!

<br/>

* ```const``` 객체가 만일 상수식으로 초기화 되었다 하더라도 컴파일 마다 런타임에 초기화 할지 안 할지 다를 수 있다.
* 그래서 정확히 컴파일 타임에 상수를 얻고자 하면 ```constexpr```를 사용하자.

<br/>

## constexpr 함수

* 기존에 컴파일 타임 상수인 객체를 만드는 함수는 TMP를 사용하여 만들었다.
* 하지만 구현된 코드를 이해하기 어렵고 반복문들은 재귀 호출의 형태로 구현해야 해서 복잡하다.

<br/>

* 이제는 함수 리턴 타입에 ```constexpr```을 사용해 조건만 맞는다면 쉽게 만들 수 있다.

```cpp
constexpr int Factorial(int n) {
  int total = 1;
  for (int i = 1; i <= n; i++) {
    total *= i;
  }
  return total;
}

template <int N>
struct A {
  int operator()() { return N; }
};

int main() {
  A<Factorial(10)> a;

  std::cout << a() << std::endl;
}
```

> 실행 결과
> 3628800

* ```Factorial(10)``` 이 컴파일 타임에 계산되어서 클래스 A 의 템플릿 인자로 들어가게 된다.

<br/>

* C++ 14 부터 아래와 제약 조건들은 ```constexpr``` 함수 내부에서 사용 불가하다.
  * ```goto``` 문 사용
  * 예외 처리 (```try``` 문; C++ 20 부터 가능하게 바뀌었다.)
  * 리터럴 타입이 아닌 변수의 정의
  * 초기화 되지 않는 변수의 정의
  * 실행 중간에 ```constexpr``` 이 아닌 함수를 호출하게 됨

<br/>

* ```constexpr``` 함수에 인자로 컴파일 타임 상수들을 전달하면, 그 반환값 역시 컴파일 타임 상수가 된다.
  * ```constexpr``` 함수에 컴파일 타임 상수가 아닌 값을 전달하면 그냥 일반 함수 처럼 작동한다.

## 리터럴 타입

* 리터럴 타입은 컴파일러가 컴파일 타임에 정의할 수있는 타입이다.
* C++에서 정의하는 리터럴 타입은 아래와 같다
  * void 형
  * 스칼라 타입 (```char```, ```int```, ```bool```, ```long```, ```float```, ```double```) 등등
  * 레퍼런스 타입
  * 리터럴 타입의 배열
  * 디폴트 소멸자를 가지고 다음 중 하나를 만족하는 타입
    * 람다 함수
    * ```Arggregate``` 타입 (사용자 정의 생성자, 소멸자가 없으며 모든 데이터 멤버들이 ```public```) 쉽게 말해 ```pair``` 같은 애들을 이야기함
    * ```constexpr``` 생성자를 가지며 복사 및 이동 생성자가 없음

<br/>

* 위 객체들만이 ```constexpr``` 로 선언되던지 ```constexpr``` 함수 내부에서 사용될 수 있다.

## constexpr 생성자

* C++14 부터 ```constexpr``` 덕분에 사용자가 직접 리터럴 타입을 만들 수 있게 되었다
* 일반적인 ```constexpr``` 함수에서 적용되는 제약조건들이 모두 적용된다.
* ```constexpr``` 생성자는 인자들이 반드시 리터럴 타입이여야 하고 해당 클래스는 다른 클래스를 상속 받을 수 없다.

```cpp
class A {
 public:
  constexpr A(int x, int y) : x_(x), y_(y) {}

  constexpr int x() const { return x_; }
  constexpr int y() const { return y_; }

 private:
  int x_;
  int y_;
};
```

* A의 생성자는 리터럴인 ```int```두 개를 인자로 받고 있다. 따라서 이는 적합한 ```constexpr``` 생성자가 된다.
* 마찬가지로 두 멤버 변수를 접근하는 함수 역시 ```constexpr`` 로 정의했다. 
  * 따라서 ```x()``` 와 ```y()``` 역시 ```constexpr``` 함수 내부에서 사용할 수 있게 된다.

<br/>
  
```cpp
class Myclass {
 public:
  constexpr Myclass(int x, int y) : x_(x), y_(y) {}

  constexpr int x() const { return x_; }
  constexpr int y() const { return y_; }

 private:
  int x_;
  int y_;
};

constexpr Myclass Add(const Myclass& v1, const Myclass& v2) {
  return {v1.x() + v2.x(), v1.y() + v2.y()};
}

template <int N>
struct A {
  int operator()() { return N; }
};

int main() {
  constexpr Myclass v1{1, 2};
  constexpr Myclass v2{2, 3};

  // constexpr 객체의 constexpr 멤버 함수는 역시 constexpr!
  A<v1.x()> a;
  std::cout << a() << std::endl;

  // Add 역시 constexpr 을 리턴한다.
  A<Add(v1, v2).x()> b;
  std::cout << b() << std::endl;
}
```

* 생성자를 ```constexpr```를 이용했기 때문에 해당 클래스의 객체를 ```constexpr```로 만들 수 있다.

<br/>

* ``` A<v1.x()> a;``` ```v1```의 ```constexpr``` 멤버 함수인 ```x``` 를 호출하였는데, ```x``` 역시 ```constexpr``` 함수이므로 위 코드는 결국 ```A<1> a``` 와 다름이 없게 된다.
* 만일 ```v1``` 이나 ```x``` 가 하나라도 ```constexpr``` 이 아니라면 위 코드는 컴파일 되지
* **constexpr 객체의 constexpr 멤버 함수만이 constexpr 을 준다.**

<br/>

* ```Add``` 함수는 마찬가지로 ```v1``` 과 ```v2``` 를 인자로 받아서 ```constexpr``` 객체를 리턴한다. 
* 이것이 가능한 이유는 ```Add``` 이 ```constexpr``` 함수 이고, ```Myclass``` 가 리터럴 타입이기 때문이다.

<br/>

## if constexpr

* C++ 표준 라이브러리의 ```<type_traits>```에서는 여러가지 템플릿 함수들을 제공하는데, 
* 이들 중 해당 타입이 포인터 인지 아닌지 확인하는 함수도 있다
* 이를 이용한 예시를 보자.

```cpp
template <typename T>
void show_value(T t) {
  if (std::is_pointer<T>::value) {
    std::cout << "포인터 이다 : " << *t << std::endl;
  } else {
    std::cout << "포인터가 아니다 : " << t << std::endl;
  }
}

int main() {
  int x = 3;
  show_value(x);

  int* p = &x;
  show_value(p);
}
```

* 컴파일시 오류가 생긴다
* ```std::is_pointer```는 전달한 인자 ```T``` 가 포인터라면 ```value``` 가 ```true``` 가 되고, 
* 포인터가 아니면 ```false``` 가 되는 템플릿 메타 함수 이다.
  * 하지만 문제는 템플릿이 인스턴스화 되면서 생성되는 코드에 컴파일이 불가능한 부분이 발생한다는 점이다.
  
```show_value(p);```

* 위 코드 경우 ```int```타입인데 ```if```문이 절대 실행되지 않음에도 불구하고 컴파일 되지 않기 때문에 오류가 발생한 것이다.

<br/>

* ```if constexpr``` 을 도입하면 깔끔히 해결된다.
* ```if constexpr```은 우선 조건이 반드시 ```bool``` 타입으로 변환될 수 있어야 하는 컴파일 타임 상수식이여야 한다.
* 해당 조건의 true와 false에 따라서 if - else가 각각 아예 컴파일이 되지 않음
  * 어떠한 타입인지 검사를 해서 해당 타입에 대해서만 어떠한 작업을 하고 아니라면 일반적인 작업을 하는 로직을 짜고 싶을 때 이용한다.

<br/>

## C++ 20

* 추가 기능들 중에 ```constexpr vector```와 ```constexpr string```가 있다.
* 이를 위해서 ```constexpr new```와 ```constexpr``` 소멸자가 추가되는데 추후에 공부하자.

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```