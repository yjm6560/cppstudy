# Item 4: 객체는 사용하기 전에 항상 초기화하자

## 왜 초기화를 해야할까?

#### 객체는 항상 초기화되지는 않는다

```c++
int x; // 0으로 초기화된다
class Point
{
  int x, y;
};
Point p; // p.x, p.y 가 0으로 초기화되었는지 아닌지 알 수 없다
```

#### 초기화 보장 여부에 따른 룰은 아주 복잡하다

초기화 발생을 보장하는 룰과 그렇지 않은 룰은 아주 복잡하다.

c++의 c 파트에서는 초기화가 런타임 코스트를 발생시키기 때문에, 초기화 발생을 보장하지 않는다.

c++의 non-c 파트에서는 초기화 발생이 보장된다.(e.g. std::vector)

#### 이런 복잡함을 피하는 가장 좋은 방법

객체 사용 전에 항상 초기화를 수행하자!

```c++
int x = 0;
const char *text = "A C-style string";

double d;
std::cin >> d;
```



## 객체는 생성자에서 제대로 초기화해주자

primitive 타입이 아닌 클래스의 객체는 생성자에서 초기화해줘야 한다.

#### 초기화는 생성자 body 에 들어가기 전에 수행되어야 한다

```c++
class PhoneNumber { ... };
class ABEntry 
{
public:
  ABEntry(const std::string &name,
          const std::string &address,
          const std::list<PhoneNumber> &phones);
private:
  std::string theName;
  std::string theAddress;
  std::list<PhoneNumber> thePhones;
  int numTimesConsulted;
};

ABEntry::ABEntry(const std::string &name,
                 const std::string &address,
                 const std::list<PhoneNumber> &phones)
{
  theName = name;
  theAddress = address; 
  thePhones = phones; 
  numTimesConsulted = 0;  
}
```

위의 예(assignment-based)는 기대하는 대로 동작하긴 하지만, **생성자 body 에서 assignment 가 이루어졌기 때문에 이건 초기화가 아니다.**

진짜 초기화는 아래처럼 **멤버 초기화 리스트**로 **생성자 body 에 진입하기 전에 수행되어야 한다**

```c++
ABEntry::ABEntry(const std::string& name, 
                 const std::string& address, 
                 const std::list<PhoneNumber>& phones)
                 : theName(name), 
                   theAddress(address), 
                   thePhones(phones),
                   numTimesConsulted(0)
{}

// user-defined type 멤버 변수들을 default constructor 로 초기화
ABEntry::ABEntry(): theName(), 
                    theAddress(), 
                    thePhones(),
                    numTimesConsulted(0)
{}
```



## 멤버 초기화 리스트를 사용해야 하는 이유

#### default constructor 호출 낭비 방지

클래스의 생성자는 멤버 변수들의 default constructor 호출한 다음, 생성자 body 의 assignment statement 들을 수행한다.

**Assignment-based 방법은 이 default constructor 호출하는 부분이 낭비된다.**

Assignment-based 방법은 assignment 를... 멤버 초기화 리스트는 복사 생성자를 사용하여 멤버 변수를 초기화한다.

#### 모든 멤버 변수를 멤버 초기화 리스트를 사용해 초기화해주자

user-defined 타입의 멤버 변수가 멤버 초기화 리스트에 없는 경우, 컴파일러는 멤버 변수의 default constructor 를 호출한다.

그렇게 되면 프로그래머는 어떤 멤버 변수가 default constructor 가 호출되고, 어떤 멤버 변수가 아닌지 기억해야 한다.

심지어 멤버 변수의 클래스에 default constructor 가 없으면 컴파일도 안 된다.

또한 const 로 선언된 멤버 변수는 당연히 멤버 초기화 리스트로 초기화해줘야 한다.

그러니까 그냥 모든 멤버 변수를 멤버 초기화 리스트로 초기화해주자.

#### 반드시 멤버 초기화 리스트만을 사용해야 할까?

많은 클래스가 여러 개의 생성자를 가지고 있어서, 생성자마다 멤버 초기화 리스트를 만드는 건 보기 싫을 수 있다.

이런 경우에는 assingment 로 초기화가 가능한 변수들은 따로 초기화 함수를 만들어 뺄 수도 있긴 하다.

초기값을 파일이나 db 에서 읽어오는 경우 특히 유용할 수 있긴 한데, 그래도~ 웬만하면 멤버 초기화 리스트를 사용하는 게 좋다.



## 멤버 초기화 순서

멤버 초기화 리스트를 사용할 때 순서에 주의해야 한다.

컴파일러 상관 없이 **반드시 멤버 변수가 선언된 순서대로 초기화를 해줘야 한다.**

물론 멤버 초기화 리스트의 순서를 바꿔도 컴파일은 되지만, 어차피 초기화 순서는 멤버변수가 선언된 순서대로 수행된다.

그러나 이건 의도치 않은 동작을 발생시킬 수 있고, 코드 읽는 사람도 헷갈릴 수 있으니 그냥 멤버 변수가 선언된 순서로 초기화를 해주자.





## Non-local static 객체의 초기화는 각각의 translation unit 에서 정해진다

#### static 객체

생성된 후로 프로세스가 종료될 때까지 살아있는 객체.

local static 객체 : 함수 안에서 static 선언된 object

non-local static 객체 : global object, namespace 에 정의된 object, class / 파일 범위 안에서 static 선언된 object

#### translation unit

하나의 object file 을 만드는 데 사용되는 소스 코드

#### Non-local static 객체의 초기화 순서에 따라 발생하는 문제

non-local static 객체를 가지고 있는 A, B 소스코드가 있다고 하자.

A 의 non-local static 객체의 초기화에서 B 의 non-local static 객체를 사용한다면?

-> Non-local static 객체의 초기화 순서는 소스코드 별로 순서를 정할 수 없기 때문에 B 의 non-local static 객체는 초기화되지 않았을 수도 있다.

###### 문제가 발생하는 예시

```c++
// FileSystem source file
class FileSystem 
{ 
public:
  ...
  std::size_t numDisks() const; 
  ...
};
extern FileSystem tfs; // non-local static 객체

// Directory source file
class Directory 
{ 
public:
  Directory(params); 
  ...
};
Directory::Directory(params) 
{
  ...
  std::size_t disks = tfs.numDisks(); 
  ...
}
Directory tempDir(params); // non-local static 객체
```

tfs 초기화 전에 tempDir 초기화가 발생한다면 당연히 큰 문제가 생길 수 있다..

#### Singleton 패턴을 사용해 non-local static 객체를 local static 객체로 바꾸기

Singleton 패턴을 사용하면 아래 예시와 같이 non-local static 객체를 local static 객체로 바꿀 수 있다.

local static 객체(함수 안에서 선언된 static 객체)는 해당 함수에 처음 접근될 때 초기화되므로 문제가 해결된다.

\+보너스1 : 사용되지 않는 non-local static 객체는 생성되지 않으므로 메모리도 절약할 수 있다..!

\+보너스2 : 함수 호출 빈도가 작다면 inline 으로 바꿔도 된다..!

```c++
class FileSystem { ... };
FileSystem& tfs() 
{
  static FileSystem fs; 
  return fs;
}
class Directory { ... };
Directory::Directory(params) 
{
  ...
  std::size_t disks = tfs().numDisks(); 
  ...
}
Directory& tempDir() 
{
  static Directory td(params); 
  return td;
}
```

## multi-threaded 시스템에서의 초기화

multi-threaded 시스템에서는 singleton 함수로의 첫 접근이 동시다발적으로 발생해 문제가 발생할 가능성이 아주 높다.

그러므로 thread 띄우기 전에 singleton 함수들을 미리 한번씩 호출해주면 초기화 문제로 발생하는 race condition 을 막을 수 있다.



## 정리

- built-in 타입 객체는 자동으로 초기화 될 때도 있고 안 될 때도 있으니 직접 초기화하자.
- 생성자에서는 assignment-based 방식보다는 멤버 초기화 리스트를 사용하고, 멤버 초기화 리스트 순서는 멤버 변수 선언 순서대로 적어두자
- translation unit 의 non-local static object 를 local static object 로 바꿔서 초기화 순서 문제를 피하자
