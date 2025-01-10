---
title:  "shared_ptr, weak_ptr" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2025-01-11
last_modified_at: 2025-01-11
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

### shared_ptr

* ```unique_ptr```와 다르게 때에 따라서는 여러 개의 스마트 포인터가 하나의 객체를 같이 소유 해야 하는 경우가 있다.
* 예를 들어서 여러 객체에서 하나의 자원을 사용하고 한다. 
* 후에 자원을 해제하기 위해서는 이 자원을 사용하는 모든 객체들이 소멸되야 하는데, 
* 어떤 객체가 먼저 소멸되는지 알 수 없기 때문에 이 자원 역시 어느 타이밍에 해제 시켜야 할 지 알 수 없게 된다.

<br/>

* ```shared_ptr```는 특정 자원을 몇 개의 객체에서 가리키는지를 추적한 다음에, 그 수가 0 이 되야만 비로소 해제를 시켜주는 방식의 포인터다.
* ```shared_ptr``` 로 객체를 가리킬 경우, 다른 ```shared_ptr``` 역시 그 객체를 가리킬 수 있다.

```cpp
std::shared_ptr<A> p1(new A());
std::shared_ptr<A> p2(p1); // p2 역시 위에서 생성된 객체 A 를 가리킨다.
```

<br/>  

* 해당 스마트 포인터는 **참조 개수(reference count)**라는게 있는데 몇개의 ```shared_ptr```가 객체를 가리키는지 알려준다.
  * 참조 개수가 0이 되어야 가리키던 객체를 해제할 수 있다.
  * ```use_count```로 현재 참조 개수가 몇개인지 확인할 수 있다. ```std::cout << p.use_count();``` 
  
<br/>  

* 여기서 해당 포인터 객체체들은 어떻게 서로 참조 객체를 동기화 하는가?
* 처음으로 실제 객체를 가리키는 ```shared_ptr```이 **제어블럭(control block)**을 동적으로 할당한다.
* 이 제어 블럭에서 서로 정보를 공유한다.
  * 해당 포인터를 생성할 때 마다 해당 제어 블럭의 위치를 공유를하여
  * 생성할 때는 제어 블록의 참조 개수를 한 개 늘리고
  * 객체가 소멸할 때는 참조 개수를 하나 줄인다.

<br/>

### std::make_shared

* 해당 포인터를 다음과 같이 평범하게 생성할 것이다.

```std::shared_ptr<A> p1(new A());```

* 이 방법은 A를 생성하기 위해 동적 할당 1번, 제어 블럭 생성을 위해 동적 할당 1번 ==> 총 2번이다.
* 동적 할당을 2번할 것을 알고 있으니 그냥 처음부터 2개를 합친 크기로 1번 할당하는게 훨씬 빠르다.

<br/>

```std::shared_ptr<A> p1 = std::make_shared<A>();```

* ```make_shared``` 함수는 ```A``` 의 생성자의 인자들을 받아서 이를 통해 객체 ```A``` 와 ```shared_ptr``` 의 제어 블록 까지 한 번에 동적 할당 한 후에 만들어진 ```shared_ptr``` 을 리턴한다다.
* ```<>```에 ```shared_ptr``` 가리킬 타입을 전달하면 알아서 해당 타입의 생성자와 제어 블럭까지 한 번에 할당한다.
* ```()```에는 해당 타입 생성자의 인자를 전달하면 된다.
  * 해당 타입의 생성자한테 완벽한 전달을 수행한다.

<br/>

### shared_ptr 생성시 주의할 점

* ```shared_ptr```를 생성할 때 인자로 객체가 아닌 주소값을 전달하면 해당 객체를 첫번째로 소유하는 ```shared_str```마냥 행동한다.

```cpp
A* a = new A();
std::shared_ptr<A> pa1(a);
std::shared_ptr<A> pa2(a);
```

* 위 예시에서 각각의 ```shared_ptr```이 서로 다른 제어 블록을 가져서 서로의 존재를 몰라 참조 카운터가 따로 체크된다.
  * 참조 개수가 서로 공유되지 않아 아직 ```shared_ptr```가가 남아있음에도 가리키는 객체를 해제할 수 도 있다.
* 그러니 ```shared_ptr```를 주소값을 통해서 생성하는 것은 위험하다!!

<br/>

### enable_shared_from_this

* 위에서 ```shared_ptr``` 주소값을 넘기지 못한다고 했는데 객체 내부에서 자기 자신을 리턴할 때는 우짜지?

```cpp
class A{
  // data
  public:
  // 생성자, 소말자 등등
  std::shared_ptr<A> get_shared_ptr() { return std::shared_ptr<A>(this); }
}
```

* ```this``` 를 사용해서 ```shared_ptr```를 리턴하면 위에서 말한 문제가 발생한다.
* ```enable_shared_from_this```를 이용해서 해결이 가능하다.

<br/>

* ```enable_shared_from_this``` 클래스에는 ```shared_from_this``` 멤버 함수가 정의되어 있다.
* 이 함수는 이미 정의되어 있는 제어 블럭을 사용해서 ```shared_ptr```을 생성한다.
  * 따라서 위 처럼 제어 블록이 또 생성되는 일을 막을 수 있다.

``` std::shared_ptr<A> get_shared_ptr() { return shared_from_this(); }```

<br/>

* 한 가지 주의할 점은 해당 함수를 사용하기 위해서는 반드시 해당 객체에 대한 ```shared_ptr```가 한 개는 있어야 한다. 
  * ```shared_from_this```는 제어 블럭을 확인만 할 뿐 새로운 제어 블럭을 만들지 않기 때문이다.

<br/>

## weak_ptr

* 해당 스마트 포인터는 ```shared_ptr```의 **순환 참조 문제**를 해결하기 위해 존재한다.

```cpp
class A : public std::enable_shared_from_this<A> {
  int *data;

 public:
  A() {
    data = new int[100];
    std::cout << "자원을 획득함!" << std::endl;
  }

  ~A() {
    std::cout << "소멸자 호출!" << std::endl;
    delete[] data;
  }

  std::shared_ptr<A> get_shared_ptr() { return shared_from_this(); }
};

int main() {
  std::shared_ptr<A> pa1 = std::make_shared<A>();
  std::shared_ptr<A> pa2 = pa1->get_shared_ptr();

  std::cout << pa1.use_count() << std::endl;
  std::cout << pa2.use_count() << std::endl;
}
```

* 객체1이 파괴 되기 위해서는 객체1을 가리키고 있는 ```shared_ptr```의 참조개수가 0이 되어야 하고
* 객체2가 파괴 되기 위해서는 객체2를 가리키고 있는 ```shared_ptr```의 참조개수가 0이되어야 한다.
  * 이러지도 저러지도 못하는 상황이 된다.
  * 해당 문제를 **순환 참조 문제**라고 한다.

<br/>

* ```weak_ptr```는 일반 포인터와 ```shared_ptr``` 사이에 위치한 스마트 포인터로, 
* 스마트 포인터 처럼 객체를 안전하게 참조할 수 있게 해주지만, 
* ```shared_ptr```와는 다르게 참조 개수를 늘리지는 않는다.
  * ```weak_ptr```를 사용하면 ```shared_ptr```가 관리하는 자원(메모리)을 참조 카운트에 영향을 미치지 않으면서 참조 타입으로 가질 수 있다.
    * 따라서 어떤 객체를 ```weak_ptr```이 가리키고 있더라도 ```shared_ptr```가 가리키지 않는다면 해당 객체는 메모리에서 소멸된다.

<br/>

* ```weak_ptr```이 가리키는 객체를 참조하기 위해서는 그냥 접급을 하지 못하고 ```shared_ptr```로 변환해서 사용해야 한다.
  * 이 때 가리키고 있는 객체가 이미 소멸되었다면 빈 ```shared_ptr```로 변환되고, 아닐경우 해당 객체를 가리키는 ```shared_ptr```로 변환된다.
  * ```weak_ptr```에 접근할 수 있다고(```weak_ptr```가 가리키고 있다고) 해서 해당 자원에 접근할 수 있다는 것을 보장할 수 없기 때문에
  * ```weak_ptr```은 할당 받은 자원을 직접적으로 사용하지 못한다.
    * 그래서 직접적인 사용을 막기 위해 ```->```, ```*```, ```get()``` 연산자는 없다.

<br/>

```cpp
class A {
  std::string s;
  std::weak_ptr<A> other;

 public:
  A(const std::string& s) : s(s) { std::cout << "자원을 획득함!" << std::endl; }

  ~A() { std::cout << "소멸자 호출!" << std::endl; }

  void set_other(std::weak_ptr<A> o) { other = o; }
  void access_other() {
    std::shared_ptr<A> o = other.lock();
    if (o) {
      std::cout << "접근 : " << o->name() << std::endl;
    } else {
      std::cout << "이미 소멸됨 ㅠ" << std::endl;
    }
  }
  std::string name() { return s; }
};

int main() {
  std::vector<std::shared_ptr<A>> vec;
  vec.push_back(std::make_shared<A>("자원 1"));
  vec.push_back(std::make_shared<A>("자원 2"));

  vec[0]->set_other(vec[1]);
  vec[1]->set_other(vec[0]);

  // 각각의 ref count 는 그대로다.
  std::cout << "vec[0] ref count : " << vec[0].use_count() << std::endl;
  std::cout << "vec[1] ref count : " << vec[1].use_count() << std::endl;

  // weak_ptr 로 해당 객체 접근하기
  vec[0]->access_other();

  // 벡터 마지막 원소 제거 (vec[1] 소멸)
  vec.pop_back();
  vec[0]->access_other();  // 접근 실패!
}
```

* ```weak_ptr```는 생성자로 ```shared_ptr``` 또는 ```weak_ptr```를 받는다.
* 또한 ```shared_ptr``` 과는 다르게 이미 제어 블록이 만들어진 객체만이 의미를 가지기 때문에, 평범한 포인터 주소값으로 ```weak_ptr``` 를 생성할 수 는 없다.

<br/>

* 앞서 말했듯이 weak_ptr 그 자체로는 원소를 참조할 수 없고, shared_ptr 로 변환해야 하는데 ```lock``` 함수를 통해 수행할 수 있다.

```std::shared_ptr<A> o = other.lock();```

<br/>

* ```shared_ptr```는 ```false```로 형변화 되기에 위 ```if```문을 활용하는 것을 볼 수 있다.

### 약한 참조 개수(weak count)

* ```shared_ptr```의 참조 개수가 0이 되고 ```weak_ptr```이 남아 있으면 해당 제어 블럭은 어떻게 될까??
* 제어 블럭을 메모리에서 해제하기 위해서는 이를 가르키는 ```weak_ptr``` 역시 0개 여야한다.
  * 왜냐하면 ```weak_ptr```가 제어 블록에서 참조 카운트가 0이라는 사실을 알지 못하기 때문이다.  
    * 제어 블록을 메모리에서 해제하기 위해서는 ```weak_ptr``` 역시 0개 여야 하고
    * 그래서 해당 ```weak_ptr```의 개수를 파악하기 위해 제어 블럭에 참조 개수와 함께 **weak count**를 기록한다.

<br/>

---

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```