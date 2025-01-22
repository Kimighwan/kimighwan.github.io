---
title:  "std::function" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-01-22
last_modified_at: 2025-01-22
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

## std::function

* C++에서 호출 가능한 모든 것을 Callable이라 한다
  * 함수가 아님에도 ()로 호출하는 것도 포함.

```cpp
struct S {
  void operator()(int a, int b) { std::cout << "a + b = " << a + b << std::endl; }
};

int main() {
  S some_obj;

  some_obj(3, 5);
}
```

* ```same_obj``` 는 함수는 아니지만 ```same_obj``` 클래스 S 의 객체이다. 
* 하지만 ```same_obj``` 는 마치 함수 처럼 ```()``` 를 이용해서 호출할 수 있다. (실제로는 ```same_obj.operator()(3, 5)```)

<br/>

* C++ 에서는 이러한 ```Callable``` 들을 객체의 형태로 보관할 수 있는 ```std::function``` 이라는 클래스를 제공한다.
* function 객체는 템플릿 인자로 전달 받을 함수의 타입을 갖는다
* 여기서 함수의 타입이라 하면, 리턴값과 함수의 인자들이다.

```cpp
int some_func1(const std::string& a) {
  std::cout << "Func1 호출! " << a << std::endl;
  return 0;
}

struct S {
  void operator()(char c) { std::cout << "Func2 호출! " << c << std::endl; }
};

int main() {
  std::function<int(const std::string&)> f1 = some_func1;
  std::function<void(char)> f2 = S();
  std::function<void()> f3 = []() { std::cout << "Func3 호출! " << std::endl; };

  f1("hello");
  f2('c');
  f3();
}
```

> 실행 결과
> Func1 호출! hello
> Func2 호출! c
> Func3 호출

<br/>

* f1의 경우 ```some_fucn1```은 ```int```를 리턴하며 ```const std::string&```을 받기에 ```<int(const std::string&)>```을 전달하여 정의했다.
* f2인 경우 단순히 함수 객체를 넘겨주고 인자로 ```char```를 받고 ```void```를 리턴하니 ```<void(char)>```  표현
* f3은 람다 함수로 리턴값, 인자값 모두 ```void```이므로 ```<void()>```으로 정의

<br/>

## 멤버 함수를 가지는 std::function

* 멤버 함수를 보관하려면 조금 이야기가 달라진다.
  * 멤버 함수 내에서 ```this```의 경우 자신을 호출한 객체를 의미하는데 그냥 ```function```에 넣게 된다면 ```this```가 무엇인지 알 수 없는 문제가 발생한다.
  * 그냥 함수를 저정하기에 어떤 객체에 대한지 정보가 없음.

```cpp
class A {
  int c;

 public:
  A(int c) : c(c) {}
  int some_func() { std::cout << "내부 데이터 : " << c << std::endl; }
};

int main() {
  A a(5);
  std::function<int()> f1 = a.some_func;
}
```

* 컴파일 오류가 난다.
* ```f1``` 을 호출하였을 때 함수의 입장에서 자신을 호출하는 객체가 무엇인지 알 길이 없기 때문에 ```c```를 참조 하였을 때 어떤 객체의 ```c``` 인지를 알 수 없다.
* 따라서 ```f1```에 ```a```에 관한 정보도 추가로 전달해야 한다.
  
<br/>

* 이를 해결할까?
* 사실 멤버 함수들은 구현 상 자신을 호출한 객체를 인자로 암묵적으로 받고있다.


```cpp
class A {
  int c;

 public:
  A(int c) : c(c) {}
  int some_func() {
    std::cout << "비상수 함수: " << ++c << std::endl;
    return c;
  }

  int some_const_function() const {
    std::cout << "상수 함수: " << c << std::endl;
    return c;
  }

  static void st() {}
};

int main() {
  A a(5);
  std::function<int(A&)> f1 = &A::some_func;
  std::function<int(const A&)> f2 = &A::some_const_function;

  f1(a);
  f2(a);
}
```

> 실행 결과
> 비상수 함수: 6
> 상수 함수: 6

<br/>

```cpp
std::function<int(A&)> f1 = &A::some_func;
std::function<int(const A&)> f2 = &A::some_const_function;
```

* 와 같이 원래 인자에 추가적으로 객체를 받는 인자를 전달해주면 된다. 
* 이 때 상수 함수의 경우 당연히 상수 형태로 인자를 받아야 하고 ```(const A&)```, 
* 반면에 상수 함수가 아닌 경우 단순히 ```A&``` 의 형태로 인자를 받으면 된다.

<br/>

* 앞서 다르게 ```&A::some_func``` 함수 이름 앞에 &가 붙어있다.
  * A::은 클래스의 멤버 함수를 나타내기 위해 사용했고
* 이는 C++의 규칙인데 멤버 함수가 아닌 모든 함수는 이름이 함수의 주소값으로 암시작 변환이 일어나지만
* 멤버함수의 경우 그렇지 않아서 명시적으로 ```&```를 사용해 주소값을 전달한다.

<br/>

---

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```


