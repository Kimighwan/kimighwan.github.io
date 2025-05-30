---
title:  "얕은 복사 && 깊은 복사" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2024-12-22
last_modified_at: 2025-04-30
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

---

# 얕은 복사

* 동적 메모리 할당에 대해서 단순한 복사는 얕은 복사를 한다.

* 평범하게 디폴트 복사 생성자를 사용한다면 원본 객체로 부터 완전히 1:1로 복사가 된다.
  * 이 때 복사된 객체는 원본 객체와 완전히 독립적이다.
  * 하지만 원본 객체의 포인터도 1:1로 복사가 되어 같은 주소를 가리키는 객체가 생성된다.
  
> 따라서 멤버 변수에 포인터나 레퍼런스 변수가 있다면 해당 주소값을 복사해 완전한 사본이 아니게 된다.
>> 같은 주소를 가르키는 포인터가 여럿 있다면 한 개의 객체가 소멸되면 해당 메모리가 해제되어 다른 객체가 가르키고 있던 데이터를 찾을 수 없게 된다.

<br/>

## 깊은 복사

* 얕은 복사와 다르게 주소를 복사하는 것이 아닌 메모리를 새로 할당해서 내용을 복사하는 방식

* 깊은 복사를 하기 위해서는 직접 **복사 생성자**를 만들어 줘야한다.
  * call by value의 경우 항상 복사 생성자가 호출된다.

```cpp
T(const T& resource)  // 복사 생성자
{
  // 새로운 메모리 할당
  // resource로 부터 내용 복사
}
```

---

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```

