# Item 23: non-member, non-friend 함수를 선호하자

### Member function과 non-member function 중 뭐가 더 좋을까?

- member function

```c++
class WebBrowser 
{ 
public:
	...
	void clearCache();
  void clearHistory();
  void removeCookies(); 
  ...
  void clearEverything(); // calls clearCache, clearHistory and removeCookies
};
```

- Non-member function

```c++
class WebBrowser 
{ 
public:
	...
	void clearCache();
  void clearHistory();
  void removeCookies(); 
  ...
};

void clearBrowser(WebBrowser& wb)
{
	wb.clearCache( ); 
  wb.clearHistory( ); 
  wb.removeCookies( );
}
```

일반적인 객체지향적 관점에서는 멤버 관련 데이터나 함수는 함께 묶어두는 것이 좋기 때문에 멤버 함수로 사용하는 것이 좋아 보인다.

**하지만 non-member && non-friend 함수로 둘 수 있는 경우에는 그렇게 하는 게 좋다.**



### non-member && non-friend 함수는 캡슐화가 더 잘 되어 있다.

Non-member && non-friend 함수는 클래스의 public 멤버 함수에만 접근할 수 있다.

즉 private 영역의 데이터 변수나 함수들에 접근하지 못해 캡슐화 측면에서 더 안정적이다.



### Non-member && non-friend 함수는 패키징 유연성에 좋다.

non-member && non-friend 함수를 사용한다는 것이 해당 함수가 다른 클래스의 멤버 함수가 될 수 없다는 의미는 아니다.

특히 namespace를 사용해서 조금 더 명확하게 사용할 수 있다.

```c++
namespace WebBrowserStuff 
{
	class WebBrowser { ... };
	void clearBrowser(WebBrowser& wb); 
  ...
}
```

이럴 경우 기능에 따라 여러 소스 파일에 나누는 것도 가능하기 때문에 컴파일 의존성이 적어진다.

```c++
// webBrowser.h
namespace WebBrowserStuff 
{
	class WebBrowser { ... };
  ...
}
// webBrowserBookmarks.h
namespace WebBrowserStuff 
{
  ...
}
// webBrowserCookies.h
namespace WebBrowserStuff 
{
  ...
}
```

Cookie 기능을 이용하지 않을 거라면 굳이 webBrowserCookies 헤더를 include 할 필요가 없다.

또한 나중에 새로운 기능을 추가하고 싶을 때도 확장성이 좋다.



### 요약

- 캡슐화, 패키징 유연성 및 확장성 측면에서 좋으므로 non-member && non-friend 함수를 사용할 수 있으면 사용해보자





