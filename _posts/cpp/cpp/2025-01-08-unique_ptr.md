---
title:  "unique_ptr" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-01-08
last_modified_at: 2025-01-08
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

## Resource Acquisition Is Initialization - RAII

* 자원 관리를 스택에 할당한 객체를 통해 수행하는 디자인 패턴.
* 예외가 발생해서 함수를 빠져나가도 ```stack unwinding```이 발생하고, 그렇지 않으면 함수가 종료될 때 소멸자들이 호출된다.
* 일반적인 포인터는 객체가 아니라 소멸자가 호출되지 않는다.
* 그렇다면 포인터를 객체로 만들어서 자신이 소멸될 때 자신이 가리키고 있는 데이터도 같이 ```delete```하면 
* 자원 관리를 스택의 객체를 통해 하게되어 좋지 않을까? 
  * 이러한 포인터를 ```smart pointer```라고 한다.

<br/>

## smart pointer

* C++11 에서는 ```unique_ptr```와 ```shared_ptr```를 지원한다.
* C++ 에서 메모리를 잘못 관리하면 크게 두 가지 문제점이 발생한다.
  * 메모리 누수
    * 위의 ```RAII``` 패턴을 사용하여 해결할 수 있다
  * 해제된 메모리를 다시 참조하는 경우
    * 문제가 발생한 이유는 만들어진 객체의 소유권이 명확하지 않아서다. 
    * 만약에, 우리가 어떤 포인터에 객체의 유일한 소유권을 부여해서, 이 포인터 말고는 객체를 소멸시킬 수 없다! 라고 한다면, 
    * 위와 같이 같은 객체를 두 번 소멸시켜버리는 일은 없을 것이다.

<br/>

### unique_ptr

* 특정 객체에 유일한 소유권을 부여하는 포인터 객체.
* 객체이기 때문에 ```RAII``` 패턴을 사용할 수 있다.

```cpp
class A {
  int *data;

 public:
  A() {
    std::cout << "자원을 획득함!" << std::endl;
    data = new int[100];
  }

  void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

  ~A() {
    std::cout << "자원을 해제함!" << std::endl;
    delete[] data;
  }
};

void do_something() {
  std::unique_ptr<A> pa(new A());
  pa->some();
}

int main() { do_something(); }
```

> 실행 결과
> 자원을 획득함!
> 일반 포인터와 동일하게 사용가능!
> 자원을 해제함!

* ```unique_ptr```를 정의하는 방법은 ```std::unique_ptr<A> pa(new A());``` 템플릿 인자로 포인터가 가리킬 클래스를 전달하면 된다.

<br/>

* 만약 ```unique_ptr```를 복사하려고 하면 어떻게 될까?

```cpp
class A {
  int *data;

 public:
  A() {
    std::cout << "자원을 획득함!" << std::endl;
    data = new int[100];
  }

  void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

  ~A() {
    std::cout << "자원을 해제함!" << std::endl;
    delete[] data;
  }
};

void do_something() {
  std::unique_ptr<A> pa(new A());

  // pb 도 객체를 가리키게 할 수 있을까?
  std::unique_ptr<A> pb = pa;
}

int main() { do_something(); }
```

* 컴파일하면 삭제된 함수를 사용한다고 오류가 뜬다.
  * ```unique_ptr```은 복사 생성자가 명시적으로 삭제됨
  * 어떠한 객체를 **유일하게** 소유해야 하기 떄문이다.
    * 그로 인해 동일 메모리를 해제하는 문제가 발생하지 않는다.

<br/>

#### unique_ptr 소유권 이전하기

* ```unique_ptr```는 복사는 되지 않지만 소유권은 이전할 수 있다.

```cpp
class A {
  int *data;

 public:
  A() {
    std::cout << "자원을 획득함!" << std::endl;
    data = new int[100];
  }

  void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

  ~A() {
    std::cout << "자원을 해제함!" << std::endl;
    delete[] data;
  }
};

void do_something() {
  std::unique_ptr<A> pa(new A());
  std::cout << "pa : ";
  pa->some();

  // pb 에 소유권을 이전.
  std::unique_ptr<A> pb = std::move(pa);
  std::cout << "pb : ";
  pb->some();
}

int main() { do_something(); }
```

> 실행 결과
> 자원을 획득함!
> pa : 일반 포인터와 동일하게 사용가능!
> pb : 일반 포인터와 동일하게 사용가능!
> 자원을 해제함!

* 이동 생성자는 소유권을 이동시킨다는 개념으로 사용이 가능하다.
* ```pa```는 가리키고 있는 주소값은 ```nullptr```이 된다.
  * 따라서 이동후 ```pa```에 접근하면 문제가 발생한다.
  * 소유권이 이전된 ```unique_ptr```를 **dangling pointer**라고 한다.
    * 이를 참조하면 런타임 오류가 발생한다.

<br/>

#### unique_ptr를 함수 인자로 전달하기

* ```unique_ptr```는 복사 생성자가 없는데 함수 인자로 어떻게 전달하지?

```cpp
class A {
  int* data;

 public:
  A() {
    std::cout << "자원을 획득함!" << std::endl;
    data = new int[100];
  }

  void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

  void do_sth(int a) {
    std::cout << "무언가를 한다!" << std::endl;
    data[0] = a;
  }

  ~A() {
    std::cout << "자원을 해제함!" << std::endl;
    delete[] data;
  }
};


void do_something(std::unique_ptr<A>& ptr) { ptr->do_sth(3); }

int main() {
  std::unique_ptr<A> pa(new A());
  do_something(pa);
}
```

> 실행 결과
> 자원을 획득함!
> 무언가를 한다!
> 자원을 해제함!

* 함수로 전달되어 제대로 동작은 하긴 한다.
* 전달 받은 ```ptr```은 레퍼런스이기에 함수가 종료되면서 객체를 파괴하지는 않는다.
* 하지만 함수 안에서도 ```ptr```에 접근을 할 수 있게 된다.
  * ```unique_ptr```의 ```get``` 함수는 실제 객체의 주소값을 리턴해준다.
  * 해당 값을 함수로 넘겨주는 방법으로 ```unique_ptr```에 접근은 하지 않지만 결국은 객체에 접근이 가능하다.
  * 일시적으로 객체에 접근하고 싶으면 해당 방법을 사용하면 된다.
* 하지만 함수 내에서 ```unique_ptr```에 접근해서 어떠한 작업을 한다면(새로운 포인터 소유) 그냥 함수 인자로 ```unique_ptr<T>&```를 넘기자
  * 대신 함수 안에서 얼마든지 조작할 수 있어 실수를 유발할 수 있어 주의가 필요하다!

<br/>

#### unique_ptr 을 쉽게 생성하기

* C++14 부터 ```unique_ptr```를 간단히 만드는 ```std::make_unique```를 제공한다.
```auto ptr = std::make_unique<T>(// 생성자 인자);```

#### unique_ptr 를 원소로 가지는 컨테이너

* 복사 생성자가 없는 특성 때문에 종종 오류를 볼 수 있다.
  * ```vector``` 의 ```push_back``` 함수는 전달된 인자를 복사해서 집어 넣기 때문에 위와 같은 문제가 발생

```cpp
int main() {
  std::vector<std::unique_ptr<A>> vec;
  std::unique_ptr<A> pa(new A(1));

  vec.push_back(std::move(pa));  // 잘 실행됨
}
```

* 명시적으로 ```vector```안으로 이동시켜줘야 한다.
* 그런데 emplace_back 함수는 전달된 인자를 perfect forwarding을 통해
* 직접 ```unique_ptr<A>``` 의 생성자에 전달 해서, ```vector``` 맨 뒤에 ```unique_ptr<A>``` 객체를 생성한다.
  * 따라서, 위에서 처럼 불필요한 이동 연산이 필요 없다.

```cpp
int main() {
  std::vector<std::unique_ptr<A>> vec;

  // vec.push_back(std::unique_ptr<A>(new A(1))); 과 동일
  vec.emplace_back(new A(1));
}
```

<br/>

---

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```