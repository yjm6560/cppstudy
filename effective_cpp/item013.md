# Item 13: 자원 관리 객체를 사용하자

c++ 은 프로그래머가 직접 메모리를 관리해야 하기 때문에 메모리 누수가 발생하지 않도록 하는 것이 중요하다

이를 위해서는 반드시 사용을 완료한 자원을 해제해주어야 한다

## 사용이 끝난 자원은 반드시 해제되는가?

```c++
class Investment
{
  ...
};

void f()
{
  Investment *pInv = createInvestment(); // createInvestment : 동적 생성된 Investment object 반환
  ...
  delete pInv; // 자원 해제
}
```

**자원을 해제하기 전에 의도치 않은 exception 이 발생한다면...? 혹은 실수로 중간에 return 해버린다면...?** -> 자원은 영원히 해제되지 않는다



## 자원 관리 객체의 소멸자에서 자원을 해제하자

함수 내에서 자원관리 객체를 생성하고, 소멸자에서 자원을 해제한다면 어떤 이유로 함수를 벗어나도 자원이 해제된다.

```c++
class ResourceManager
{
public:
  ResourceManager(Investment *apInv)
  {
    pInv = apInv;
  }
  ~ResourceManager()
  {
    delete pInv;
  }
private:
  Investment* pInv;
};
```

이를 위해 유용하게 사용되는 것이 **스마트 포인터** 



## 스마트 포인터란?

스마트 포인터의 소멸자에서 스마트 포인터가 가리키는 객체의 delete 함수를 자동으로 호출

```c++
void f()
{
  std::unique_ptr<Investment> pInv(createInvestment());
  ...
  // delete 필요 없음
}
```



## 자원 관리 객체를 사용에 있어서 중요한 특징

- 자원 획득 후 즉시 자원 관리 객체에 넘겨줘야 한다(RAII(Resource Acquisition Is Initiallization) 개념)

- 자원 관리 객체는 소멸자에서 자원의 해제를 보장해야 한다



## 스마트 포인터의 종류

#### std::unique_ptr

자원의 소유권을 **단 하나의 자원 관리 객체**가 가질 수 있는 스마트 포인터

복사 시에 원본 객체를 null 로 만든다

#### std::shared_ptr

RCSP(reference-counting smart pointer) 방식의 스마트 포인터

**자원을 가리키는 다른 객체들이 하나도 없어지면** 자원을 해제하는 스마트 포인터

```c++
void f()
{
  std::shared_ptr<Investment> a(createinvestment()); // reference-count = 1
  std::shared_ptr<Investment> b(a);                  // reference-count = 2
  ...
}
```

f 함수 종료 후

1. b 자원 해제 -> reference-count = 1

2. a 자원 해제 -> reference-count = 0 -> 자원 해제 



## 스마트 포인터의 한계

책에서 스마트 포인터는 배열 자원을 해제할 수 없다고 한다

스마트 포인터의 소멸자에서 delete 로 자원을 해제해주기 때문...

하지만 **사실 스마트 포인터로 배열 자원을 해제해줄 수 있다**

#### auto_ptr 대신 unique_ptr 을 사용하자

auto_ptr 은 c++11 부터 사용하지 말라고 권고 됐고, c++17 부터는 아예 사라진 개념

auto_ptr 대신 unique_ptr 을 쓰면 배열에도 가능하다

```c++
std::unique_ptr<int[]> up(new int[10]);
```

또 STL container 에 unique_ptr 을 넣는 것도 가능하다(auto_ptr 은 안 됨)

#### shared_ptr 에서 배열 자원 관리

shared_ptr 에서는 deleter 를 따로 명시해주는 게 가능하다

```c++
std::shared_ptr<int> sp(new int[10], std::default_delete<int[]>());
```

#### shared_ptr 의 순환 참조 문제

```c++
class A
{
public:
  std::shared_ptr<A> sp;
};
std::shared_ptr<A> sp1(new A());
std::shared_ptr<A> sp2(new A());
// 순환 참조 발생
sp1->sp = sp2;
sp2->sp = sp1;

sp1.reset(); // 그래도 sp2->sp 가 sp1 을 가리키므로 reference-count > 0 이 됨
```

weak_ptr 을 사용하면 reference-count 를 증가시키지 않는다

```c++
class A
{
public:
  std::weak_ptr<A> sp;
};
std::shared_ptr<A> sp1(new A());
std::shared_ptr<A> sp2(new A());

sp1->sp = sp2;
sp2->sp = sp1;

sp1.reset(); // reference-count = 0 이 되고 정상 메모리 해제
```



## 정리

- 자원 누수를 막기 위해 생성자에서 자원을 획득하고 소멸자에서 해제하는 RAII 객체를 사용하자
- 복사 시 auto_ptr 은 원본 객체를 null 로 만들고 shared_ptr 은 원본 객체를 null 로 만들지 않아 더 직관적이다