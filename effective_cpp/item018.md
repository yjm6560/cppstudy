# item 18: 인터페이스는 제대로 쓰기는 쉽게, 잘못 쓰기는 어렵게 만들자

사람들이 인터페이스를 잘못 사용하는 데는 인터페이스 설계자도 일부 책임이 있다.

그러므로 사용자가 인터페이스를 제대로 쓰기는 쉽게, 잘못 쓰기는 어렵게 만들어야 한다.

## 간단한 인터페이스 예시

- 헷갈리기 쉬운 인터페이스 예시

```c++
class Date
{
public:
    Date(int month, int day, int year);
    ...
};

Date d(3, 30, 1995); // correct
Date d(30, 3, 1995); // incorrect
Date d(3, 40, 1995); // incorrect
```

- 파라미터 순서를 잘못 쓰기 어렵게 만든 인터페이스 예시

```c++
struct Day
{
    // explicit 적용하여 implicit type conversion 방지
    explicit Day(int d) : val(d)
    {
    }
    int val;
};

struct Month
{
    explicit Month(int m) : val(m)
    {
    }
    int val;
};

struct Year
{
    explicit Year(int y) : val(y)
    {
    }
    int val;
};

class Date
{
public:
    Date(const Month& m, const Day& d, const Year& y);
    ...
};

Date d(30, 3, 1995); // error
Date d(Day(30), Month(3), Year(1995)); // error
Date d(Month(3), Day(30), Year(1995)); // okay
```

- 파라미터 값을 잘못 쓰기 어렵게 만든 예시

```c++
class Month
{
public:
    static Month Jan(){return Month(1);}
    static Month Feb(){return Month(2);}
    ...
private:
    explicit Month(int m);
};

Date d(Month::Mar(), Day(30), Year(1995));
```



## 웬만하면 기본 타입의 인터페이스와 맞추자

특별한 이유가 없다면 **일관성 유지**를 위해 기본 타입의 인터페이스와 동작을 맞추는 게 좋다.

```c++
if (a * b = c) ... // == 을 = 으로 잘못 쓰는 경우...
```

a, b, c 타입이 int 였으면 에러가 발생했을 것이다.

user-defined 클래스에서도 똑같이 에러가 발생하도록 해주는 게 좋다.

헷갈리면 그냥 int 동작 방식과 맞춰주는 게 좋다!



## 사용자가 외워야 잘 쓸 수 있는 인터페이스는 좋지 않다.

```c++
Investment* createInvestment();
```

item 13 에서 본 `createInvestment()` 함수는 동적 생성된 Investment 객체를 반환해주기 떄문에 Investment 객체 사용이 끝나면 사용자가 반드시 자원 해제를 해줘야 한다.

이걸 잊어버려서 누수가 발생하면 당연히 사용자 책임이지만 인터페이스에서 그런 일이 없도록 해주면 더 좋다.

```c++
std::shared_ptr<Investment> createInvestment();
```

이렇게 바꿔주면 사용자가 신경쓸 필요가 없어진다.

#### 만약 delete 가 아닌 다른 자원 방식의 자원 해제가 필요하다면?

```c++
void getRidOfInvestment();
```

자원을 해제하는 위와 같은 함수를 제공할 수 있지만, 사용자가 실수로 delete 를 사용할 가능성도 높기 때문에 이 자원 해제 함수를 외우고 있지 않으면 실수하기가 쉽다.

shared_ptr 을 사용한다면 deleter 도 따로 명시해줄 수 있기 때문에, **사용자가 어떻게 자원을 해제해줘야 하는지 외울 필요도 없다.**

결국 `createInvestment()` 함수는 아래와 같이 작성하는 게 좋다.

```c++
std::shared_ptr<Investment> createInvestment()
{
    std::shared_ptr<Investment> retVal(static_cast<Investemnt*>(0), getRidOfInvestment);
    retVal = ...;
    return retVal;
}
```



#### Cross-DLL 문제

객체를 생성할 때 특정 DLL 의 new 를 사용하고, 소멸시킬 때는 다른 DLL 의 delete 를 사용하게 되면  new/delete 짝이 실행되는 DLL 이 달라져서 꼬일 수 있다. -> 런타임 에러 발생

shared_ptr 은 기본적으로 생성 시 사용한 DLL 의 delete 를 사용하도록 만들어졌기 때문에 cross-DLL 문제를 막을 수 있다.

스마트 포인터를 쓰면 물론 런타임 비용이 아주 조금 늘 수는 있지만 크리티컬하지는 않기 때문에, 사용자의 실수를 줄일 수 있도록 shared_ptr 를 쓰자.



## 정리

- 좋은 인터페이스는 제대로 쓰기 쉽고, 잘못 쓰기는 어렵게 만들어야 한다.
- 좋은 인터페이스를 설계하는 방법은 다른 인터페이스들과 일관성을 유지하는 게 좋다.
- 사용자 실수를 방지하기 위해 새로운 타입 만들기, 타입에 대한 연산 제한하기, 객체가 가질 수 있는 값에 제한 걸기, 자원 관리를 인터페이스 내부적으로 해주기가 있다.
- shared_ptr 은 deleter 를 따로 지정하는 게 가능하여 cross-DLL 문제를 막아주는 등의 이점이 있으니 웬만하면 사용하도록 하자.