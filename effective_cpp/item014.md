# Item 14: 자원 관리 측면에서 복사를 잘 생각해보자

## 스마트 포인터는 resource handler 로 사용하기 적절할까?

스마트 포인터는 heap-based 자원 관리에 좋다.

하지만 모든 자원이 heap-based 인 건 아니므로, 결국 resource handler 가 필요로 해진다.

## resource handler 의 복사 문제

- resource handler 를 사용하는 행복해보이는 예제

```c++
class Lock 
{
public:
  explicit Lock(Mutex *pm) 
  : mutexPtr(pm)
  {
    lock(mutexPtr); // acquire resource
  }
  ~Lock()
  {
    unlock(mutexPtr); // release resource
  }
private:
  Mutex *mutexPtr; 
};
```

```c++
void f()
{
  Mutex m;
  Lock ml(&m);
  ...
}
```

만약 Lock object를 복사하면 어떻게 될까?

```c++
Mutex m;
Lock ml1(&m);
Lock ml2(ml1);
```

ml1.mutextPtr 과 ml2.mutextPtr 이 값이 가리키는 객체가 동일해진다.

-> ml1, ml2 의 소멸자에서 자원 해제를 시도하므로 heap-based 자원으로 치면 **double free 문제 발생** 할 수 있다.



## resource handler 의 복사 방법

#### 1. 복사 금지

많은 경우에 RAII object 의 복사를 허락하지 않는다(특히 mutex).

Synchronization primitive(platform에서 제공하는 간단한 동기화 메커니즘)를 복사하는 게 거의 허용되지 않기 때문

복사를 막고 싶다면 Uncopyable 을 private 상속받자!

```c++
class Lock: private Uncopyable
{
public:
  ...
};
```

#### 2. Reference-count 사용하기

자원을 사용 중인 resource handler object가 소멸될 때까지 기다려야 할 때도 있다.

이럴 때 RAII object를 복사하고 reference-count 를 증가시키면 된다.

shared_ptr 는 reference-count 가 0 이 될 때 소멸자 대신 호출할 함수, deleter를 정할 수 있다.

- deleter는 auto_ptr 에는 없다.
- deleter 는 optional 한 shared_ptr 생성자의 두 번째 parameter

```c++
class Lock
{
public:
  explicit Lock(Mutext *pm) // init
  :mutextPtr(pm, unlock)  // 두 번째 파라미터 unlock func를 deleter로 지정
  {
    lock(mutexPtr.get());
  }
private:
  std::shared_ptr<Mutex> mutextPtr;
};
```

#### 3. 그냥 복사하기

resource-managing class 를 자원 사용 완료 후 자원을 해제하기 위해 사용하고, 복사는 원하는 만큼 하려고 할 수도 있다.

이때 resource-managing object를 복사하려면 **깊은 복사**를 수행해야 한다.

##### e.g. string 객체 복사

string 을 이루는 문자들은 Heap 에 존재하고, 복사 시에는 heap 공간을 가리키는 포인터까지 모두 복사 -> 깊은 복사

#### 4. 소유권 넘겨주기

아주 가끔 resource 소유권을 한 객체가 가지게 하고, 그 객체가 복사되면 그 소유권을 넘겨주고자 할 수도 있다.

auto_ptr, unique_ptr 이 복사되는 방식.

```c++
std::unique_ptr<int> up2(new int);
std::unique_ptr<int> up1(std::move(up2));
```



## 정리

- RAII object 복사는 관리하는 resource의 복사를 어떻게 할지에 따라 정해진다.
- 일반적인 RAII class의 복사는 복사를 금지하거나 참조 카운팅을 해주는 정도다. 하지만 다른 것들도 가능하기는 하다.
