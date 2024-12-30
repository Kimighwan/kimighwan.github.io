---
title:  "상속" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2024-12-26
last_modified_at: 2024-12-30
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

---

# 상속

* 상속을 통하여 객체지향 프로그래밍에서 추구하는 실제 객체의 추상화를 더 효과적으로 할 수 있다.
* 그렇다고 객체지향으로 프로그래밍을 하자! 가 아닌 활용을 하여 프로그래밍을 하자!

<br/>

* 상속에 두 가지 관계를 표현할 수 있다.

<br/>

1. is-a 관계
* 아래 예시) 좀비는 몬스터이다.
``` cpp
class Enemy { // 모든 몬스터 공통적인 특징 };
class EnemyA : public Enemy { // 특정 몬스터만의 기능능};
```

<br/>

1. has-a 관계
```cpp
 class Enemy { // 체력 // 공격력 // 속도 ... 몬스터만의 정보 };
  class EnemyA {   
    private:
      // 무슨 데이터들들
    public:
        EnemyA(Enemy data); // 데이터들 초기화화
};
```
 
<br/>

---

## 다중 상속
* 한 클래스에서 여러개의 클래스를 상속 받는 것.

* 궁금한 점. 생성자의 호출 순서는 어떻게 될까?
    * 그저 상속하는 순서에만 좌우된다. 
```... class A : public B, public C // B클래스 -> C클래스 -> A클래스```

* 주의점! 상속하는 클래스에 같은 변수가 있으면 어떤걸 써야할지 모른다.
* 또 주의할 점은 다이아몬드 상속에서 중복으로 받는 정보가 있을 수 있다.
    * 이를 해결하기 위해 중간 클래스에서 상속시 접근 지시자 뒤에 virtual를 붙인다.
    * 이것이 가상 상속이다.

``` cpp
 class Monster{};
 class Zombie : public virtual Monster {};
 class Slime : public virtual Monster {};
 class ZombieSlime : public Zombie, public Slime {};
 ```

<br/>

* 다이아몬드 상속 문제를 가상 상속으로 해결한다면 손자 클래스에서 조상 클래스 생성자를 직접 호출해야 한다.
* 직접 하지 않으면 컴파일러가 자동으로 해주지만, 생성자에서 멤버 변수를 초기화 하는 행위가 있다면 필수적이다.

<br/>

* C#은 다중 상속을 지원하지 않는다.

<br/>

---

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```
