# Item 10: 할당 연산자에서 *this를 반환하게 해라

## Assignment operator의 return 값

assignment는 오른쪽에서 왼쪽으로 chain을 이루며 동작

```c++
int x, y, z;
x = y = z = 15;  // chain of assignments
 
x = ( y = ( z = 15 ) );
```

assignment operator는 left-hand argument를 반환(일종의 convention)

프로그래머가 설계한 class에도 assignment operator가 사용된다면 이 convention을 따르는 게 좋다.

```c++
class Widget
{
public:
  ...
  Widget& operator=(const Widget& rhs) // return type is a reference to the current class
  {
    ...
    return *this;   // return the left-hand object
  }
  ...
};
```

단순한 assignment operator 외에도 +=, -= 등의 모든 assignment operagor에서 left-hand object를 반환해주는 게 좋다.

```c++
class Widget
{
public;
  ...
  Widget& operator+=(const Widget& rhs) //the convention applies to +=, -=, *=, etc.
  {
    ...
    return *this;
  }
  Widget& operator=(int rhs) // it applies even if the operator's parameter type is unconventional
  {
    ...
    return *this;
  }
  ...
};
```



## assignment operator 에서 *this 반환 안 하면?

assignment operator에서 left-hand object를 반환하지 않는다고 컴파일에서 에러가 발생하거나 하지는 않음

그러나 모든 built-in type과 string, vector 등 standard library에 속한 모든 타입들이 이 convention을 따르기에 특별한 이유가 없다면 따르도록 하자



## 요약

- assignment operator는 *this의 참조값을 반환하도록 하자
