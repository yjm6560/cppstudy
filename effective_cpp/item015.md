# item 15: 자원 관리 클래스의 raw resource 에 접근할 수 있는 방법을 제공하자

raw resource 를 직접 건드리지 않으면 좋겠지만 많은 기존 api 들이 이미 자원을 직접적으로 참조하도록 되어 있다.

```c++
std::shared_ptr<Investment> pInv(createInvestment());
int daysHeld(const Investment *pi);
int days = daysHeld(pInv); // error : Investment* != std::shared_ptr<Investment>
```

그러므로 자원 관리 클래스에서 필요에 따라 raw resource 를 접근할 수 있는 방법을 제공해야 한다.



## 스마트 포인터를 사용하여 raw resource 접근하기

item 13 에서 배운 스마트 포인터를 사용하여 raw resource 에 접근하고, raw resource 가 가진 멤버 함수도 호출할 수 있다.

#### get() 함수를 통해 raw resource 반환하기

get() 함수를 사용하여 raw resource 를 반환해줄 수 있다.

```c++
int days = daysHeld(pInv.get()); // OK
```

#### raw resource 의 멤버 접근하기

잘 만들어진 스마트 포인터라면 `operater-> ` 나 `operator*` 를 제공하므로 이걸 사용해서 raw resource 의 멤버에 접근할 수 있다.

```c++
class Investment // raw resource
{
public:
  bool isTaxFree() const;
  ...
};

Investment* createInvestment();

std::shared_ptr<Investment> pi1(createInvestment());

bool taxable1 = !(pi1->isTaxFree()); // operator-> 사용하여 raw resource 접근
...
std::unique_ptr<Investment> pi2(createInvestment());
bool taxable2 = !((*pi2).isTaxFree()); // operator* 사용하여 raw resource 접근
...
```



## User-defined 자원 관리 클래스에서의 raw resource 접근

#### get() 함수를 통해 raw resource 반환하기

```c++
FontHandle getFont();

void releaseFont(FontHandle fh);

class Font // FontHandle 을 관리하는 자원 관리 클래스
{
public:
  explicit Font(FontHandle fh) : f(fh)
  {
  }
  ~Font()
  {
    releaseFont(f);
  }
  FontHandle get() const  // explicit 변환 함수로 raw resource 접근 제공
  {
    return f;
  }
private:
  FontHandle f; // raw resource
};
```

```c++
void changeFontSize(FontHandle f, int newSize);

Font f(getFont());
int newFontSize;
...
changeFontSize(f.get(), newFontSize); // Font -> FontHandle 로 변환 후 전달
```

#### implicit 변환 함수로 매끄럽게 하기

raw resource 를 직접 접근하기 위해 매번 get 함수를 호출해서 번거롭다 하면 implicit 변환 함수를 제공할 수도 있다.

```c++
class Font
{
public:
  ...
  operator FontHandle() const // implicit 변환 함수
  {
    return f;
  }
};

Font f(getFont());
int newFontSize;
...
changeFontSize(f, newFontSize);
```

다만 주의해야 할 점은 **implicit 변환 함수 때문에 의도치 않은 동작이 이루어질 수 있다는 것이다.**

```c++
Font f1(getFont());
...
FontHandle f2 = f1; // Font 객체를 복사하고 싶었지만 f1 이 FontHandle 로 타입 변환되고 FontHandle 이 복사되어 버렸다...
```

implicit 변환 함수는 편하지만 잘못 쓰일 가능성도 높기 때문에 **대부분의 경우에는 get() 함수를 사용하는 것이 좋다..!**



## raw resource 에 접근할 수 있게 하는 게 캡슐화에 반하는 일인가?

자원 관리 클래스에서 관리하는 raw resource 를 직접 반환하여 조작이 가능하도록 하는 게 객체지향 설계 원칙 중 하나인 캡슐화에 위배되는 것처럼 보일 수 있다.

틀린 말은 아니지만 **RAII 클래스의 목적은 데이터 은닉이 아닌 자원의 해제**이기 때문에 반드시 캡슐화를 제공해야 하는 것은 아니다.



## 정리

- raw resource 를 직접 접근해야 하는 기존 api 들이 많기 때문에 RAII 클래스를 만들 때는 raw resource 에 접근할 수 있는 방법을 제공해야 한다.
- raw resource 로의 접근은 explicit, implicit 변환을 통해 가능하다. explicit 은 안전하고, implicit 은 편리하므로 상황에 따라 적절한 변환을 사용하자.
