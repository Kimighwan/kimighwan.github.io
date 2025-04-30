---
title:  "perfect forwarding, std::forward" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-01-06
last_modified_at: 2025-01-06
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

## move

* 우측값 레퍼런스를 받는 이동 생성자를 통해 복사가 없는 이동이 가능했다.
* 만약 좌측값도 이동을 시키고 싶다면 어떻게 해야할 까?
    * 흔한 swap함수에서 임시 저장 변수를 한 개 만들어거 두 데이터의 내용을 swap하는데 그냥 서로 주소값만 바꿔주면 된다.
    * 하지만 이동 생성자는 ```&&```을 사용하므로 좌측값이 사용이 안 된다.

<br/>

* C++11 부터 <utility> 라이브러리에서 좌측값을 우측값으로 바꾸어주는 move 함수가 제공된다.

```cpp
class A {
 public:
  A() { std::cout << "일반 생성자 호출!" << std::endl; }
  A(const A& a) { std::cout << "복사 생성자 호출!" << std::endl; }
  A(A&& a) { std::cout << "이동 생성자 호출!" << std::endl; }
};

int main() {
  A a;

  std::cout << "---------" << std::endl;
  A b(a);

  std::cout << "---------" << std::endl;
  A c(std::move(a));
}
```

* 실행 결과

```
일반 생성자 호출!
---------
복사 생성자 호출!
---------
이동 생성자 호출!
```

* ```std::move``` 함수가 인자로 받은 객체를 우측값으로 변환해서 리턴하여 ```복사 생성자```가 아닌 ```이동 생성자```가 호출되었다.
* 함수 이름 처럼 이동을 하는 것이 아닌 그저 우측값으로 캐스팅만 한다. 

<br/>

```cpp
template <typename T>
void my_swap(T &a, T &b) {
  T tmp(a);
  a = b;
  b = tmp;
}
```

```cpp
template <typename T>
void my_swap(T &a, T &b) {
  T tmp(std::move(a));
  a = std::move(b);
  b = std::move(tmp);
}
```

* ```T tmp(std::move(a));```를 통해서 ```tmp```라는 임시 객체를 ```a```로 부터 이동 생성했다
* 이동 생성이기 때문에 복사보다 훨씬 바르게 수행된다.
* 물론 위에서 특정 타입에 대한 연산이기에 대입 연산자를 오버로딩 해줘야 한다.
* ```a = std::move(b);``` 같은 연산은 아래와 같이 이동 대입 연산자를 따로 만들어 줘야 한다.

```cpp
A& A::operator=(A&& a) {
    // 어떠한 작업 수행
    // 데이터를 복사해주고
    // a는 nullptr로 초기화화
  return *this;
}
```

* 이동 생성자와 비슷하게 간단하다.
* 한 가지 중요한 점은 데이터가 이동되는 것은 **이동 생성자**나 **이동 대입 연산자**를 호출할 때 수행된다.
  * ```std::move```를 사용한 시점이 아니다.
  * 이동 대입 연산자가 없다면 일반적인 대입 연산자가 오버로딩 되어 느린 복사가 수행된다.

<br/>

* 일반 대입 연산자나, 이동 대입 연산자 모두 자기 자신에 대해 대입의 경우 그냥 바로 ```*this```를 전달하고 ```return``` 한다.

<br/>

```cpp
class A {
 public:
  A() { std::cout << "ctor\n"; }
  A(const A& a) { std::cout << "copy ctor\n"; }
  A(A&& a) { std::cout << "move ctor\n"; }
};
class B {
 public:
  A a_;
};
```

* 위 ```A``` 객체를 만들어서 ```B```의 객체 안으로 이동시키고 싶다면 어떻게 해야할 까?
* 이동 시키기 위해 ```B``` 생성자에서 ```std::move```를 사용해볼까?

<br/>

```cpp
class A {
 public:
  A() { std::cout << "ctor\n"; }
  A(const A& a) { std::cout << "copy ctor\n"; }
  A(A&& a) { std::cout << "move ctor\n"; }
};

class B {
 public:
  B(const A& a) : a_(std::move(a)) {}

  A a_;
};

int main() {
  A a;
  std::cout << "create B-- \n";
  B b(a);   // A 복사 생성자 호출
}
```

* 여기서는 복사 생성자를 이용한다.
* 문제는 ```B(const A& a)``` 생성자에서 ```const A&```라서 ```std::move(a)``` 타입은 ```const A&&``` 가 된다.
* 그런데 ```A```의 생성자에는 ```const A&```와 ```A&&``` 두 개 밖에 없다.
  * 따라서 여기선 컴파일러가 ```const A&``` 를 택한다. 따라서 복사 생성자가 호출...

<br/>

* 그렇다면 ```B``` 생성자에서 아예 우측값을 받아야 한다.

```cpp
class B {
 public:
  B(A&& a) : a_(a) {}

  A a_;
};

int main() {
  A a;
  std::cout << "create B-- \n";
  B b(std::move(a));    // A 복사 생성자 호출
}
```

* 그럼에도 여전히 ```A```의 복사 생성자가 호출된다.
*  ```B(A&& a) : a_(a) {}``` 여기서 ```a```를 우측값 레퍼런스로 받았지만 그 자체로는 좌측값이기에 ```A``` 생성자는 복사 생성자가 호출된다.

<br/>

```cpp
class A {
 public:
  A() { std::cout << "ctor\n"; }
  A(const A& a) { std::cout << "copy ctor\n"; }
  A(A&& a) { std::cout << "move ctor\n"; }
};

class B {
 public:
  B(A&& a) : a_(std::move(a)) {}

  A a_;
};

int main() {
  A a;
  std::cout << "create B-- \n";
  B b(std::move(a));
}
```

* 이제서야 ```A```의 이동 생성자가 이용되면서 ```B``` 객체안에서 ```A``` 객체가 만들어진다.

<br/>

## perfect forwarding

* **rvalue** 레퍼런스를 전달을 통해 함수 안에서 사용하기 위해 이동 생성자와 이동 대입 연산자를 사용했다
* 그런데 함수 안에서는 해당 변수 표현식이 **lvalue**가 된다.
  * **rvalue** 레퍼런스를 다른 곳으로 전달하면서 사용하기 위해서는 **rvalue**가 유지되어야 한다.
  * 이렇게 우측값으로 유지하며 전달하는 것을 **perfect forwarding**이라고 한다
    * 그것을 ```std::move()```를 활용하여 수행한다.
      * 그냥 ```(T&&)```로 캐스팅하는 것이 옛날에는 안 되었는데, 지금은 컴파일러거 업그레이드 되면서 된다.

<br/>

* 함수 안에서 ```lvalue```로 작동하는게 문제이니 ```rvalue```로 바꾸는 ```std::move()```를 쓰면 그만 아닌가?
  * 문제는 move에서 ```lvalue```와 ```rvalue```를 구별을 못하는 상황이 있다.
* 일반적으로 ```T&&```에는 ```rvalue```만 들어가고 ```T&```는 ```lvalue```만 들어간다.
  * 하지만 템플릿에서 상황이 다르다.
  * ```rvalue``` 레퍼런스를 받는 템플릿이 있다면, ```rvalue```는 당연하고 ```lvalue```까지 받아서 인스턴스화한다.
    * 이것을 **universal reference(보편 참조)**라고 한다

<br/>

* 그렇다면 ```T&&```를 받는 템플릿에 좌측값이 들어온다면 ```T```타입은 어떻게 해석될까?
  * C++11의 **reference collapsing rule(레퍼런스 겹침 규칙)**에 따라 ```T```의 타입을 추론한다.

```cpp
typedef int& T;
T& r1;   // int& &; r1 은 int&
T&& r2;  // int & &&;  r2 는 int&

typedef int&& U;
U& r3;   // int && &; r3 는 int&
U&& r4;  // int && &&; r4 는 int&&
```

* 간단하게 ```&``` 는 1 이고 ```&&``` 은 0 이라 둔 뒤에, OR 연산을 하면 된다.

<br/>

* 그래서 템플릿에 좌측값 우측값 둘다 들어와도 작동하는데 ```ravalue```의 경우 내부에서 ```lvalue```로 바뀌니 ```std::move```를 사용할 것이다
* 그런데 ```lvalue```가 들어온다면 우리는 ```lvalue```로써 사용하고 싶은데 ```std::move```로 인해 원치 않는 ```rvalue```로 사용될 수 있다.
  * 이 상황을 해결하기 위해 ```std::forward```를 사용한다.

<br/>

## std::forward

```cpp
void A(int& i)
{
    cout << "l-value" << endl;
}

void A(int&& i)
{
    cout << "r-value" << endl;
}

template<typename T>
void Tset( T&& t)
{
    A(t);
}

int main()
{
    int i = 1;
    Test(i);  // r-value를 받는 템플릿이 없어서 오류가 생길 것 같지만...
    Test(3);  
}
```

* ```Test(i);```에서 ```l-value```를 사용하지만 정상적으로 템플릿이 동작한다. 
  * 이것이 위에서 말한 **universal reference(보편 참조)**
* 그리고 rvalue 성질이 계속 유지되지 않아서 ```Test(i);```과 ```Test(3);``` 모두 ```void A(int& i)```가 호출된다.

<br/>

- 그렇다면 rvalue 성질을 그대로 유지하기 위해 ```std::move()```사용하면 문제가
  * 위에서 말했듯이 템플릿을 통하여 값을 받으면 구별을 못하게되고 여기서 ```move```를 쓰면 ```l-value```로 쓰고 싶던 값이 ```r-value```로 원치 않게 사용된다.
  * 위 예시의 템플릿에서 rvalue를 받는다고 무작정 생성자 호출시 ```std::move()```를 사용하면 lvalue로 기대되는 상황에서도 rvalue로 쓰이게 된다.

<br/>

* 위 상황을 해결하기 위해서 ```std::forward<T>()```가 생겼다
* 무조건 템플릿에서만 사용해야한다.

<br/>

* ```A(t);```에서 ```A(std::forward<T>(t));```바꾸어 사용하면 드디어 원하는 방식으로 동작한다.
* 사용할때 템플릿 파라미터를 명시해줘야 정상적으로 작동한다.

<br/>

---

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```