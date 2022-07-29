# Item 27: 타입 캐스팅을 최소화하자

c++에서의 타입 캐스팅은 c, java, c# 보다 더 위험하다.

## 옛날 캐스팅 스타일보다는 c++ 스타일의 캐스팅을 사용하자

### 옛날 캐스팅 스타일

(T)expression or T(expression)3

### c++ 스타일의 캐스팅 종류

##### 1. const_cast\<T>(expression)

object에 붙은 const를 없애준다.

##### 2. dynamic_cast\<T>(expression)

주로 상속 관계에서 안정적인 다운캐스팅을 위해 사용한다.

런타임 코스트가 크다.

##### 3. reinterpret_cast\<T>(expression)

low-level 캐스팅을 수행한다(e.g. pointer to int).
구현환경에 의존적이어서 이식성이 떨어진다.

##### 4. static_cast\<T>(expression)

implicit conversion을 강제하기 위해 사용하는 일반적인 캐스팅.

e.g. int to double, non-const obj to const obj.

### c++ 스타일 캐스팅을 사용하자

알아보기도 쉽고, 캐스팅 목적에 따라 캐스팅 방법이 세분화되기 때문에 컴파일러가 에러를 발견하기도 수월하다.

옛날 캐스팅 스타일을 사용하는 거의 단 한 가지 경우는 아래처럼 함수의 인자로 넘길 explicit 생성자를 호출하려고 할 때다.

```c++
class Widget
{
public:
	  explicit Widget(int size);
	  ...
};

void doSomething(const Widget& w);

// 둘은 똑같은 동작을 수행
doSomething(Widget(15)); // 옛날 스타일(T(expression) 형식) 캐스팅
doSomething(static_cast<Widget>(15)); // C++ 스타일 캐스팅
```



## c++ 캐스팅의 비용

많은 사람들이 타입 캐스팅은 컴파일러에게 이 타입을 다른 타입으로 처리하라고 알려주는 용도로만 사용된다고 알고 있지만, 실제로는 런타임에 발생하는 비용이 큰 경우가 많다.

```c++
int x, y;
...
double d = static_cast<double>(x) / y;
```

대부분의 아키텍쳐에서 int와 double을 처리하는 방식이 다르기 때문에 위와 같은 경우 런타임에 실행되는 double 연산을 위한 코드를 생성하게 된다.

### 하나의 object를 가리키는 pointer 값이 다를 수 있다

```c++
class Base { ... };
class Derived: public Base { ... };
Derived d;
Base* pb = &d; // Derived* -> Base*로 암묵적 형변환
```

포인터의 offset을 Derived\*에 적용하여, 실제 base\*의 포인터값을 구하는 동작이 런타임에 동작한다.

즉, 형변환에 따라 하나의 object를 가리키는 pointer 값이 달라질 수 있다(특히 다중상속 상황에서는 무조건).

참고 : https://www.sysnet.pe.kr/2/0/11164



### 객체 복사 때문에 발생하는 문제

##### 부모 클래스의 가상함수를 자식 클래스에서 재정의할 때 부모 클래스의 함수를 호출하는 요구사항을 받았다고 가정하면...

```c++
class Window
{ 
//base class
public: 
    virtual void onResize() 
    {
        ...
    }; 
    ...
}; 

class SpecialWindow : public Window 
{ 
//derived class
public: 
    virtual void onResize() 
    { 
        static_cast<Window>(*this).onResize();
        ...
    } 
    ...
};
```

static-cast 를 수행하면서 복사된 Window 객체가 만들어지고, 복사된 객체의 onResize 함수를 호출하게 된다.

그러면 onResize 함수를 호출한 SpecialWindow 의 상태는 변하지 않는다..

변하는 건 static_cast를 통해 복사된 Window 객체의 상태.

##### 타입 캐스팅을 하지 않도록 하자..

그냥 기본 클래스의 함수를 호출하도록 하면 해결할 수 있다.

```c++
class SpecialWindow : public Window 
{ 
public: 
    virtual void onResize() 
    { 
        Window::onResize();
        … 
    }
    ...
};
```



## dynamic_cast 의 비용

dynamic_cast는 일반적으로 부모 클래스의 포인터로 자식 클래스의 함수를 호출하고 싶을 때 사용하게 되는데... 런타임에 아주 느리게 동작한다.

그러므로 웬만하면 dynamic_cast를 피해보도록 하자.



#### 자식 클래스의 포인터를 컨테이너에 저장하자

##### dynamic_cast를 사용하는 예시

```c++
class Window 
{
    ...
};

class SpecialWindow: public Window 
{
public:
  	void blink();
  	...
};

typedef std::vector<std::tr1::shared_ptr<Window>> VPW; // 부모 클래스를 담는 컨테이너
VPW winPtrs;
...
for (VPW::iterator iter = winPtrs.begin();iter != winPtrs.end();++iter) 
{
    // dynamic_cast 사용 -> 안 좋은 예시
  	if (SpecialWindow *psw = dynamic_cast<SpecialWindow*>(iter->get())) 
    {
        psw->blink();
    }
}
```

##### dynamic_cast 사용하지 않는 예시

```c++
typedef std::vector<std::tr1::shared_ptr<SpecialWindow> > VPSW; // 자식 클래스를 담는 컨테이너
VPSW winPtrs;
...
for (VPSW::iterator iter = winPtrs.begin();
     iter != winPtrs.end();
     ++iter)
{
	  (*iter)->blink( );  
}
```

물론 이 방법으로는 부모 클래스를 상속 받는 모든 자식 클래스를 하나의 컨테이너에 저장하는 게 불가능하다...



#### 부모 클래스에 가상 함수를 만들어버리자

부모클래스에 가상 함수를 만들어놓으면, 부모 클래스 포인터로 자식 클래스의 함수를 호출할 수 있다!

```c++
class Window 
{ 
public:
  	virtual void blink() {}
  	... 
};

class SpecialWindow: public Window 
{ 
public:
  	virtual void blink() 
    { 
        ... 
    }
	  ...
};

typedef std::vector<std::tr1::shared_ptr<Window>> VPW;
VPW winPtrs;
...
  
for (VPW::iterator iter = winPtrs.begin();iter != winPtrs.end();++iter)
{
    (*iter)->blink();
}
```



#### 가장 피해야 할 방식, cascading dynamic_casts

```c++
class Window 
{ 
  ... 
};
... 
std::vector<std::tr1::shared_ptr<Window> > VPW;
VPW winPtrs;
...
for (VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) 
{
  	if (SpecialWindow1 *psw1 = dynamic_cast<SpecialWindow1*>(iter->get())) 
    { 
        ... 
    }
  	else if (SpecialWindow2 *psw2 = dynamic_cast<SpecialWindow2*>(iter->get())) 
    { 
        ... 
    }
	  else if (SpecialWindow3 *psw3 = dynamic_cast<SpecialWindow3*>(iter->get())) 
    { 
        ... 
    }
    ... 
}
```

이렇게 폭포식 dynamic_cast를 사용하도록 구현해놓으면, 굉장히 코드가 커지고 느려지고 망가지기 쉽다..

예를 들어 자식 클래스가 하나 추가된다면? -> 또 코드를 우겨넣어야 한다...



## 요약

- 웬만하면 타입 캐스팅을 피하자. 특히 dynamic_cast는!
- 어쩔 수 없이 타입 캐스팅을 써야하면 client가 본인들 코드에 타입캐스팅을 사용하지 않도록 함수 내부에 숨겨두자.
- 옛날 캐스팅 방식보다는 c++ 캐스팅 방식을 사용하도록 하자.