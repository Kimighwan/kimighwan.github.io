---
title:  "random & time" 

categories:
  - Cpp

toc: false
toc_sticky: false

date: 2024-12-28
last_modified_at: 2024-12-28
---

[모두의 코드](https://modoocode.com/135) 내용을 공부하고 정리한 내용입니다.
{: .notice}

<br/>

---

# random

* c언어의 난수는 내가 보통 생각하던 완전한 무작위 난수가 아닌 의사 난수이다.
    * 기초값으로 이미 정해진 과정에 따라 그냥 만들어진다.
    * 보통 시드값을 time(NULL)로 조절하는데 이 단위는 초단위이기 때문에 변화 폭이 좁다
        * 시간대가 비슷하거나 같이 돌아가는 프로그램은 같은 수를 만든다.
* 난수가 정확히 균등하지 않으며 rand()는 선형 합동 생성기 (Linear congruential generator) 이라는 알고리즘을 바탕으로 구현
    * 좋다고 말하지 못하는 품질의 난수를 만든다.

<br/>

* C++의 <random> 라이브러리를 사용해보자.

<br/>

```cpp
#include <iostream>
#include <random>
int main() {
  // 시드값을 얻기 위한 random_device 객체 생성.
  std::random_device rd;

  // random_device 객체를 통해 난수 생성 엔진 객체를 초기화 한다.
  std::mt19937 gen(rd());

  // 0 부터 99 까지 균등하게 나타나는 난수열을 생성하기 위해 균등 분포 정의.
  std::uniform_int_distribution<int> dis(0, 99);

  for (int i = 0; i < 5; i++) {
    std::cout << "난수 : " << dis(gen) << std::endl;
  }
}
```

<br/>

### 시드값

* C의 경우 time()을 통해 시드값을 지정했지만, C++에서는 random_device를 통해 시드값을 제공한다.
    * random_device는 비결정적 난수를 생성하는 난수 생성기인데
    * 예측 불가능한 물리적 소스에서 획득한 난수를 의미한다.
        * 하드웨어 장치를 이용하여 난수를 생성하는데 (예를 들어 장치 드라이버의 noise)
        * 주변 환경과 상호작용을 하여 만들기에 의사 난수보다 속도가 느리다.

<br/>

* 그래서 시드값으로 난수 엔진을 초기화 하는데 사용하고, 이후 난수열은 난수 엔진으로 생성하는 것이 적합하다.

<br/>

### 난수 엔진

* 난수 엔진에 사용되는 알고리즘은 c++에서 4가지가 있다.

<br/>

1. inear_congruential_engine(C++11)
* 선형 합동 생성기(LCG)를 기반인 난수 엔진, 생성된 결과는 정수만이 생성된다.
* 현재 상태를 저장하기 위한 메모리 사용량이 가장 적다.
    * 생성된 난수(정수) 또는 아직 생성안되어 초기 시드값을 담은 정수 2가지.
* 간단하지만 가장 빠르거나 높은 풀질을 가지는 것은 아니다.
 
<br/>

2. mersenne_twister_engine(C++11)
* 소프트웨어 기반 난수 생성기 중에서 가장 품질이 좋고 속도가 빠른 엔진이다.
* Mersenne Twister 알고리즘 기반반
* 하지만 암호학적으로 안전하지 않은 부호 없는 정수 난수를 간격에 생성한다.
 
<br/>
  
3. subtract_with_carry_engine(C++11)
* carry를 이용한 뺴기 알고리즘을 사용한다
 
<br/>

4. philox_engine(C++26)
* 카운터 기반 난수 엔진.

<br/>

* 위의 4가지 엔진을 이용한 다양한 난수 생성기가 있다.
* 위 예시에서 사용된 mt19937엔진은 mersenne twister 알고리즘을 사용하고
* 생성되는 난수들 간의 상관관계가 매우 작아 시뮬레이션에 많이 사용.
    * 하지만 객체의 크기가 크다.(2KB 이상)

<br/>

### 분포


* 위 예시 처럼 엔진을 만들었다고 난수를 생성할 수 있는게 아님
* C++은 어떤 범위에서 난수를 뽑을지 분포를 정의해야 한다.
    * 분포는 일정 범위에 숫자가 분포된 방식을 표현하는 수학 공식이다.

<br/>

* 여러가지 분포 객체들이 존재한다.
* 원하는 방식에 맞는 템플릿을 선택하면 된다.
* 아래 2가지만 알아보자.

<br/>

1. 균등 분포 (Uniform distribution)
```std::uniform_int_distribution<int> dis(0, 99);```
* 생성자로 범위를 지정하며 객체를 생성.
* 그 후 분포 객체에 엔진을 인자로 전달한다.
  
<br/>

2. 정규 분포
``` std::normal_distribution<double> dist(0, 1);```
* 객체를 만들며 생성자 인자로 평균과 표준 편차를 정해준다.
   
<br/>
   
---
   
# chrono

* chrono는 크게 3가지로 구성됨
  * 현재 시간을 알려주는 시계
  * 시간의 간격을 나타내는 duration
  * 특정 시간을 나타내는 time_stamp
  
<br/>

* 실제 시계 처럼 지금 몇 시 이렇게 이야기 해주는 것이 아니다.
* 지정된 시점으로 부터 몇 번의 틱(tick)이 발생 하였는지 알려주는 time_stamp 객체를 리턴
  * time_stamp 객체는 clock의 시작점과 현재 시간의 duration을 보관하는 객체다.

<br/>

* 각 시계 마다 정밀도가 다르기 때문에 각 clock 에서 얻어지는 tick 값 자체는 조금씩 다르다.
    * 가령 system_clock이 1초에 1tick이라면 high_resolution_clock은 0.00000001 초 마다 1 tick

<br/>

* 예시로 아래 난수 생성 속도 측정을 보자.

```cpp
#include <chrono>
#include <iomanip>
#include <iostream>
#include <random>
#include <vector>
int main() {
    std::random_device rd;
    std::mt19937 eng(rd());
    std::uniform_int_distribution<> dist(0, 1000);
    
    for (int total = 1; total <= 1000000; total *= 10) {
        std::vector<int> random_numbers;
        random_numbers.reserve(total);
        std::chrono::time_point<std::chrono::high_resolution_clock> start = std::chrono::high_resolution_clock::now();
        for (int i = 0; i < total; i++) { random_numbers.push_back(dist(eng)); }
        std::chrono::time_point<std::chrono::high_resolution_clock> end = std::chrono::high_resolution_clock::now();
        auto diff = end - start; // C++ 17 이전 
        // C++ 17 이후 ==> std::chrono::duration diff = end - start; 
        std::cout << std::setw(7) << total << "개 난수 생성 시 틱 횟수 : " << diff.count() << std::endl;
    }
}
```

<br/>

```cpp
std::chrono::time_point<std::chrono::high_resolution_clock> start = std::chrono::high_resolution_clock::now();
std::chrono::time_point<std::chrono::high_resolution_clock> end =  std::chrono::high_resolution_clock::now();
``` 

* chrono 라이브러리의 경우 다른 표준 라이브러리와는 다르게 객체들이 std::chrono 이름 공간 안에 정의되어 있습니다.
* clock에는 현재의 time_point를 리턴하는 static 함수 now()가 정의되어 있다.
    * clock에 맞는 time point를 리턴한다.
      * 위 예시는 ```std::chrono::time_point<std::chrono::high_resolution_clock>```를 리턴한다.
<br/>

## time_point
* 특정한 시점을 표현하는 클래스로 에포크(epoch, 기준시간)을 기준으로 측정한 duration을 저장.
* 기준시간(에포크)는 특정 clock의 시작 시간이 된다. 
* 그래서 time point 객체를 생성하기 위해 특정 clock을 <>에 전달해야 한다.

<br/>

## duration_cast
* duration에는 count 멤버 함수가 있는데 시간 차이 동안 몇 번의 틱이 발생했는지 알려준다.
  * 하지만 우리에게 의미 있는 정보는 틱이 아니라 실제 시간이므로 duration_cast를 이용해 변경해줘야 한다.

<br/>

* duration 객체를 받아서 원하는 duration으로 캐스팅 가능하다.

```ch::duration_cast<std::chrono::microseconds>(diff).count()```

* ```std::chrono::microseconds```는 미리 정의된 duration 객체.
  * count값은 duration이 몇 마이크로초 인가를 나타낸다.
    * 이외에 nanoseconds, milliseconds, seconds, minutes, hours도 있다
 
 <br/>
 
## 현재 시간
* 그런데 안타깝게 C++에는 현재시간을 간단하게 다룰 수 있는 클래스가 없어 c함수에 의존해야 한다.

```cpp
#include <chrono>
#include <ctime>
#include <iomanip>
#include <iostream>

int main() {
  auto now = std::chrono::system_clock::now();
  std::time_t t = std::chrono::system_clock::to_time_t(now);
  std::cout << "현재 시간은 : " << std::put_time(std::localtime(&t), "%F %T %z")
            << '\n';
}
```

* 먼저 system_clock 에서 현재의 time_point 를 얻어온 후에, 날짜를 출력하기 위해서 time_t 객체로 변환.
  * time이라 함은 크게 time_t 타입과 tm구조체가 있다.
    * time_t 타입은 시간을 나타내는데 1970년 1월 1일 00:00 UTC (유닉스 타임)이후 경과된 초(정수값)를 나타낸다. (time_t 타입은 ctime 헤더 파일에 정의)
    * tm 구조체 이는 우리가 알아보기 쉽게 변수로 나눠진 구조체다.
    ![](https://velog.velcdn.com/images/gogion/post/859d3880-11bb-4aa1-9f20-339c23d53d3c/image.PNG)

<br/>

* system_clock에는 time_point와 time_t 타입의 C스타일 시간을 상호 변환하는 static 함수가 2개 있다

1. to_time_t()는 인수로 전달 받는 time_point를 time_t로 리턴
2. form_time_t()는 인수로 전달 받는 time_t를 time_point로 리턴

* 그래서 time_point를 time_t로 변환하기 위해  ```std::chrono::system_clock::to_time_t(now);```를 사용
  * 그렇게 반환한 값을 tm구조체에 있는 원하는 형태로 바꾼다.
   * 그런데 time_t 타입은 마냥 정수로만 이루어진 숫자여서 알아볼 수  없다.
        * tm 구조체로 바꿔서 우리가 알아보기 쉽게 만들자.

<br/>

* time_t에는 tm 구조체로 변환하는 gmtime와 localtime 함수가 있다
  * 반대로 struct tm 형식을 time_t 형식으로 변환하는 mktime(mkgmtime, timegm)이 있다.

<br/>

*  C++ 2019 기준으로 gmtime 과 localtime 은 deprecate(컴파일 경고) 되었다.
   * localtime_s(tm 객체 주소, time_t 객체 주소);
     * 시스템의 타임존에 맞는 시간
   * gmtime_s(tm 객체 주소, time_t 객체 주소);
     * UTC(세계협정시간)
 
<br/>

* 마지막으로 문자열로 바꿔주는 함수
  * char* ctime(const time_t* pTime); time_t 객체를 넣으면 그대로 요일 월 날짜 시간 분 초 년도 를 출력함
  * char* asctime(const struct tm* pTm); 위와 같지만 tm객체를 넣어야함
    * 반환형 : "요일 월 일 시간:분:초 연도" ("Www Mmm dd hh:mm:ss yyyy")

<br/>

---

[<random>](https://en.cppreference.com/w/cpp/header/random)
[ctime](https://en.cppreference.com/w/cpp/header/ctime)

<br/>

```
개인 공부 기록용 블로그입니다.
틀린 부분 있으다면 지적해주시면 감사하겠습니다!!
```