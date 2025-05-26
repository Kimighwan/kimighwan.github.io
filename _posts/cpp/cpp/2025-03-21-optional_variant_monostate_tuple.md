---
title:  "std::optional, std::variant, std::monostate, std::tuple" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-03-21
last_modified_at: 2025-03-21
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

## std::optional - C++17 이상

- <potional>

```cpp
std::string GetValueFromMap(const std::map<int, std::string>& m, int key) {
	auto itr = m.find(key);
	if (itr != m.end()) {
		return itr->second;
	}
	return std::string();
}

int main() {
	std::map<int, std::string> data = { {1, "hi"}, {2, "hello"}, {3, "hiroo"} };
	std::cout << "맵에서 2 에 대응되는 값은? " << GetValueFromMap(data, 2)
		<< std::endl;
	std::cout << "맵에서 4 에 대응되는 값은? " << GetValueFromMap(data, 4)
		<< std::endl;
}
```
- 실행하면 존재하지 않는 키 '4'에 대해서 그냥 아무것도 대응되는 값을 출력안 할 뿐 프로그램은 잘 작동한다.
- 그런데 key에 대응하는 값이 빈 문자열이라면 ket가 존재하는지 않하는지 구별할 수 없다.
- 아래와 같이 대응되는 값 말고 실제로 있는지 없는지 체크를 하는 값까지 같이 반환을 해서 해결할 수 있다.

```cpp
std::pair<std::string, bool> GetValueFromMap(
  const std::map<int, std::string>& m, int key) {
  auto itr = m.find(key);
  if (itr != m.end()) {
    return std::make_pair(itr->second, true);
  }

  return std::make_pair(std::string(), false);
}
int main() {
  std::map<int, std::string> data = {{1, "hi"}, {2, "hello"}, {3, "hiroo"}};
  std::cout << "맵에서 2 에 대응되는 값은? " << GetValueFromMap(data, 2).first
            << std::endl;
  std::cout << "맵에 4 는 존재하나요 " << std::boolalpha
            << GetValueFromMap(data, 4).second << std::endl;
}
```

- 그런데 맵에 키가 존재 하지 않을 때 디폴트 객체를 리턴해야 하지만 아래 문제점이 있다.
  - 객체의 디폴트 생성자가 정의되어 있지 않을 수 도 있고
  - 객체를 디폴트 생성하는 것이 매우 오래 걸릴 수 도 있다

- 간단하게 값이 있고 없음을 표현할 수 있으며 있을 때만 값을 뽑을 수 있으면 좋겠는데....

<br/>

* C++17 표준 라이브러리에 추가된 **optional**로 간단하게 표현할 수 있다.
  * 쉽게 원하는 값을 보관하거나 or 아무것도 보관 안하거나 2가지 상태를 가진다.

* ```std::optional<>``` 템플릿 인자로 보관하고 싶은 객체 타입을 전달
* 그렇게 만들어진 ```optional``` 객체는 해당 타입 데이터를 갖던가 없던가 2개의 상태가 된다.

```cpp
std::optional<std::string> GetValueFromMap(const std::map<int, std::string>& m, int key) 
{
  auto itr = m.find(key);
  if (itr != m.end()) 
  {
    return itr->second;	// 존재한다면 해당 값을 리턴
  }

  return std::nullopt; // nullopt 는 <optional> 에 정의된 객체로 비어있는 optional 을 의미한다.
}

int main() {
  std::map<int, std::string> data = {{1, "hi"}, {2, "hello"}, {3, "hiroo"}};
  std::cout << "맵에서 2 에 대응되는 값은? " << GetValueFromMap(data, 2).value()
            << std::endl;
  std::cout << "맵에 4 는 존재하나요 " << std::boolalpha
            << GetValueFromMap(data, 4).has_value() << std::endl;
}
```

> 실행결과
> 맵에서 2 에 대응되는 값은? hello
> 맵에 4 는 존재하나요 false

<br/>

- ```optional```에 보관하는 타입을 받는 생성자가 정의되어 있어서 위와 같이 그냥 리턴하면 ```optional```객체가 반환된다
- 이 때 가장 큰 장점은 객체를 보관하는 과정에 동적 할당이 발생하지 않는다.

<br/>

* 만약 비어있는 ```optional``` 객체를 리턴하고 싶다면 ```std::nullopt``` 객체를 리턴하면 된다.
  * 빈 ```optional```객체를 나타낸다.

<br/>

* optional 객체에 들어있는 객체에 접근하고 싶으면 멤버 함수인 ```value()```를 사용하면된다.
  * 그리고 ```*``` 연산자로 데이터 멤버에 접근 가능하다.(역참조) ```*GetValueFromMap(data, 2)```

<br/>

- 주의할 점은 만약 ```optional```이 가지고 있는 객체가 없다면 ```std::bad_optional_access``` 예외를 던진다.
- 따라서 반드시 객체에 접근하기 전 값을 가지는지 체크를 해야한다
  - 값이 있는지 없는지 확인할 때는 멤버 함수인 ```has_value()``` 사용.
    - ```optional``` 객체 자체에 ```bool``` 캐스팅 연산자가 있어서 if문 같은 곳에 편하게 사용.

<br/>

## 레퍼런스를 가지는 std::optional

- ```std::optional```의 한 가지 단점으로는 일반적인 방법으로는 레퍼런스를 포함할 수 없다는 점. 
  - 예를 들어서 아래와 같이 레퍼런스에 대한 ```optional``` 객체를 정의하고 한다면

```cpp
class A {
 public:
  A() { std::cout << "디폴트 생성" << std::endl; }

  A(const A& a) { std::cout << "복사 생성" << std::endl; }
};

int main() {
  A a;

  std::optional<A&> maybe_a = a;
}
```

- 레퍼런스를 가질 때는 ```std::reference_wrapper```를 사용해서 레퍼런스 처럼 동작하는 ```wrapper```객체를 정의해야 한다.

```cpp
class A {
 public:
  int data;
};

int main() {
  A a;
  a.data = 5;

  // maybe_a 는 a 의 복사복이 아닌 a 객체 자체의 레퍼런스를 보관하게 된다.
  std::optional<std::reference_wrapper<A>> maybe_a = std::ref(a);

  maybe_a->get().data = 3;

  // 실제로 a 객체의 data 가 바뀐 것을 알 수 있다.
  std::cout << "a.data : " << a.data << std::endl;
}
```

- ```std::reference_wrapper```는 레퍼런스가 아니라 일반적인 객체이기 때문에 ```optional```에 전달할 수 있다.
- ```reference_wrapper```를 ```get()``` 함수를 통해서 레퍼런스 하고 있는 객체를 얻어올 수 있다.
- 대신 ```reference_wrapper``` 객체를 생성하기 위해서는 ```std::ref``` 함수를 사용해야 한다.

<br/>

## std::variant C++17 이상상

- <variant>

- ```std::variant```는 ```one-of``` 를 구현한 클래스이다. 
- 즉 컴파일 타임에 정해진 여러가지 타입들 중에 한 가지 타입의 객체를 보관할 수 있는 클래스 입니다.
  - 런타임에 어떤 타입인지 검사할 필요가 없다.

- 물론 공용체(union) 을 이용해서 해결할 수 도 있겠지만, 공용체가 현재 어떤 타입의 객체를 보관하고 있는지 알 수 없기 때문에 실제로 사용하기에는 매우 위험하다.

```cpp
// v 는 이제 int
std::variant<int, std::string, double> v = 1;

// v 는 이제 std::string
v = "abc";

// v는 이제 double
v = 3.14;
```

- 먼저 ```variant```를 정의할 때 포함하고자 하는 타입들을 명시해야 한다.
- 만약 초기화 없이 사용한다면 첫 번째 인자의 타입인 ```int```의의 디폭트 생성자가 호출되어 ```0```이 들어간다.
- 만약 생성하는 타입들이 디폴트 생성자가 없다면 컴파일 오류가 생긴다.

<br/>

- ```variant```는 ```optional```과 비슷하게 객체의 대입 시에 어떠한 동적 할당도 발생하지 않는다.
  - 따라서 굉장히 작은 오버헤드로 객체들을 보관할 수 있다.
- 다만 ```variant``` 객체 자체의 크기는 나열된 가능한 타입들 중 가장 큰 타입의 크기를 따라간다.

<br/>

* 그럼 ```variant```에서 원하는 값을 어떻게 얻냐?

<br/>

* 먼저 해당 하는 값의 타입이 몇번 째 인자가 저장되어 있는지 확인하는 ```index()``` 멤버 함수가 있다.
  * 0부터 시작
* 또는 그냥 타입 자체를 알고 싶으면 ```std::holds_alternative``` 함수를 사용하면 된다.
  * ```std::holds_alternative<int>(a)``` 해당 값의 타입이 맞으면 ```true```를 반환 아니면 ```false``` 

<br/>

* 원하는 값을 얻을려면 ```std::get<>()``` 함수를 사용.
  * ```<>```에 뽑고자 하는 타입을 쓰든지 아니면 해당 타입의 ```index```를 전달하면된다.
  * ```()```에는 ```variant```객체를 전달.
  
<br/>

* ```get```으로 값을 가져 올 때 저장된 값과 다른 타입을 호출하면 예외가 발생한다.
* 타입이 일치하지 않을 경우를 대비하여 ```std::get_if```를 사용할 수 있다.
  * 타입이 맞으면 해당 값을 포인터로 반환하고 아니라면 ```nullptr```을 반환한다.

<br/>

* 여기서 한 가지 알 수 있는 점은 ```varinat```가 보관하는 객체들은 타입으로 구분된다는 점
  * 따라서 ```variant```를 정의할 때 같은 타입을 여러 번 쓰면 컴파일 오류가 난다.

## std::monostate

- ```variant```에 아무 것도 들고 있지 않은 상태를 표현하고자 싶다면 해당 타입으로 ```std::monostate```를 사용하면 된다.
  - 디폴트 생성자가 있어서 초기화를 안할 때 첫 번째 타입으로 지정할 수 있다.
 
<br/>

## std::tuple C++11 이상

* 서로 다른 타입들의 묶음을 간단하게 다루는 ```std::tuple```
  
```cpp
#include <iostream>
#include <string>
#include <tuple>

int main() {
	std::tuple<int, double, std::string> tp;
	tp = std::make_tuple(1, 3.14, "hi");
	std::cout << std::get<0>(tp) << ", " << std::get<1>(tp) << ", " << std::get<2>(tp) << std::endl;
}
```
* ```std::tuple<>``` 보관하고 싶은 타입을 ```<>```에 넣어주면 정의하는 방법은 끝이다.
* ```vartiant```와 다르게 같은 타입이 들어가 있어도 문제가 없다.
* ```tuple```객체를 초기화를 위해 ```make_tuple```함수를 이용한다.

<br/>


* 값에 접근하는 방법은 ```std::get<>()```으로 ```<>```에 몇 번째 타입에 접근할지, ```()```에는 ```tuple```객체를 넣어주면 된다.
* ```<>```에 타입 자체를 넣어서 뽑을 수 있지만 해당 타입이 없거나 2개 이상이면 예외가 발생한다.
  * 2개 이상 중에 무엇을 가져와야 할지 몰라서
- 또는 ```tie```라는 함수를 이용할 수 있다 ```std::tie(x,y,z) = tupleValue;```
  - tuple의 데이터가 각각 ```x, y, z```에 들어간다.

<br/>

* 두 개 이상의 반환값 있으면 포인터나 참조를 이용하거나 구조체를 만들어 전달하는 불편함이 있었다.
* tuple을 이용한다면 반환값을 몇개이던지 쉽게 표현할 수 있다.

<br/>

## Structured binding C++17 이상

* 구조체, 컨테이너, 배열 등 어떠한 원소들의 모임에서 변수에 원소를 쉽게 가져오도록 도와준다.
 
```cpp
std::tuple<int, std::string, bool> GetStudent(int id) 
{
	if (id == 0) { return std::make_tuple(30, "철수", true); }
	else { return std::make_tuple(28, "영희", false); }
}

int main() {  
	auto student = GetStudent(1);
	auto [age, name, is_male] = student;
	std::cout << "이름 : " << name << std::endl;
	std::cout << "나이 : " << age << std::endl;
	std::cout << "남자 : " << std::boolalpha << is_male << std::endl;
}
```

```auto [age, name, is_male] = student;```
- ```auto```에 ```&```혹은 ```&&``` 도 가능하고, ```[]``` 안에 ```tuple```의 원소를 받는 객체들을 넣어주면 된다.
  - 그냥 ```auto```를 사용하면 복사가 일어나고 참조를 하기 위해 ```auto&```를 사용하면 된다.

<br/>

* [여기에서](https://en.cppreference.com/w/cpp/language/structured_binding) Structured binding이 가능한 3가지 경우를 말하고 있다. 

1. array

```cpp
int arr[3] = { 1, 2, 3 };
auto [a, b, c] = arr;
```

2. tuple-like type
- ```tuple``` 같은 타입 표현식이 ```std::tuple_size<E>::value``` 처럼 생긴 친구들은 말한다.

```cpp
std::map<int, std::string> m = {{3, "hi"}, {5, "hello"}};
for (auto& [key, val] : m) { std::cout << "Key : " << key << " value : " << val << std::endl; }
```

3. data members
- 구조체나 클래스 객체에 접근하는 것인데 겍체의 필드가 모두 접근 가능해야 하며 그 중에서 ```non-static``` 데이터 멤버에 접근가능하다.

<br/>

[optional](https://en.cppreference.com/w/cpp/utility/optional)
[std::nullopt](https://en.cppreference.com/w/cpp/utility/optional/nullopt_t)
[value_or](https://en.cppreference.com/w/cpp/utility/optional/value_or)
[variant 참고](https://mycodings.fly.dev/blog/2023-03-03-study-and-guide-of-c-17-std-variant)
[std::holds_alternative](https://en.cppreference.com/w/cpp/utility/variant/holds_alternative)
[tuple](https://en.cppreference.com/w/cpp/utility/tuple)
[Structured binding](https://en.cppreference.com/w/cpp/language/structured_binding)
[공부한 내용 복습](https://modoocode.com/135)

<br/>
  
```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```

