---
title:  "함수 객체" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2024-12-31
last_modified_at: 2024-12-31
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

---

# 함수 객체

* 함수는 아니지만 함수 처럼 작동하는 객체 Function Object => Functor
  * 함수 처럼 작동하는 객체 ==> 객체를 함수처럼 작동하게 만든다.
  * 이를 위해 operator() 연산자를 오버로딩해야한다.(호출 연산자)
* 함수 객체를 호출하면 객체.operator()(인자값) 형태로 호출하지만 중간에 .operator가 생략
  * 이를 암묵적 호출이라 한다. 반면 위처럼 표현을 온전히 하는것은 명시적 호출이다.

```cpp
class Compare
{
public:
	int operator()(int a, int b)
	{
		return a > b;
	}
};
int main()
{
	Compare com;
 
	cout << "compare(10, 20): " << com(10, 20) << endl;
	return 0;
}
```

* 위 처럼 그냥 사용하지 않고 보통 함수의 인수로 전달될 때 사용된다.

<br/>

* 왜 사용하나?
1. 일반 함수와 달리 호출 연산자 이외 데이터를 저장할 변수를 만들어 필요한 속성, 상태를 포함할 수 있다.
2. 함수객체의 생성과 실행 시점이 다르다.
3. 함수 객체는 타입이므로, 템플릿 인수로 사용될 수 있다.

<br/>

* 일반 함수는 호출시 바로 실행되지만 함수 객체는 생겅한다고 실행되는게 아니다.
  * 함수 객체를 함수에 전달해서 사용이 가능하다.
   
```cpp
template <typename T>
void sort(T& t) {
  for (int i = 0; i < t.size(); i++) {
    for (int j = i + 1; j < t.size(); j++) {
      if (t[i] > t[j]) {
        t.swap(i, j);
      }
    }
  }
}

template <typename T, typename Comp>
void bubble_sort(T& t, Comp& comp) {
  for (int i = 0; i < t.size(); i++) {
    for (int j = i + 1; j < t.size(); j++) {
      if (!comp(t[i], t[j])) {
        t.swap(i, j);
      }
    }
  }
}

struct Comp1 {
  bool operator()(int a, int b) { return a > b; }
};

struct Comp2 {
  bool operator()(int a, int b) { return a < b; }
};

int main(){
    Comp1 comp1;
    Comp2 comp2;
    
    sort(나만의 컨테이너 객체, comp1);  // 내림차순
    sort(나만의 컨테이너 객체, comp1);  // 오름차순

}
```

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```