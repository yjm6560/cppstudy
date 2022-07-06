# Item 23: 타입 변환이 모든 매개변수에 적용되어야 한다면 비멤버 함수를 선언하자

모든 매개변수에 타입 변환이 제공되어야 한다면 어떻게 함수를 선언해야 할까?

멤버 함수도 있고, 비멤버 비프렌드 함수도 있고, 비멤버 프렌드 함수도 있다.

## 예제로 사용할 유리수 클래스

```c++
class Rational
{
public:
    Rational(int numerator = 0,   // implicit constructor
             int deonminator = 1);
    int numerator() const;
    int denominator() const;
private:
   ...
};
```

## 멤버 함수로 선언할 경우

```c++
class Rational
{
public:
    ...
    const Rational operator*(const Rational& rhs) const;
};

Rational oneEighth(1, 8);
Rational oneHalf(1, 2);

Rational result = oneHalf * oneEighth; // ok
result = oneHalf * 2; // ok
result = 2 * oneHalf; // error
result = oneHalf.operator*(2); // ok
result = 2.operator*(oneHalf); // error
result = operator*(2, oneHalf); // error
```

컴파일에 성공하는 경우(=Rational 의 `operator*` 를 사용하는 경우)에는 컴파일러가 매개변수가 Rational 객체인 걸 알고 2(위 예시에서의 정수) 값을 Rational 클래스로 변환한다. Rational class 의 constructor 가 explicit 선언되지 않았으므로 암묵적으로 변환 가능하다.

하지만 컴파일에 실패하는 경우(=2의 `operator*` 를 사용하는 경우)에는 컴파일러가 매개변수로 뭘 받아야 할지 모른다. -> 컴파일 에러

## 비멤버 함수로 선언할 경우

```c++
class Rational
{
...
};

const Rational operator*(const Rational& lhs,
                         const Rational& rhs)
{
    return Rational(lhs.numerator() * rhs.numerator(),
                    lhs.denominator() * rhs.denominator());
}

Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2; // ok
result = 2 * oneFourth; // ok
```

이렇게 비멤버 함수로 바꿔주면 하나만 Rational 객체여도 컴파일러가 `operator*` 를 짐작해서 컴파일에 성공할 수 있다.

## 비멤버 프렌드 함수로 선언하면?

프렌드 함수는 웬만해서는 사용하지 않는 것이 좋다.

위의 예에서도 굳이 private 변수에 접근하지 않고 구현이 가능하기 때문에 프렌드일 필요가 없다.

**"멤버 함수여서는 안 된다."는 말이 "프렌드 함수여야 한다." 는 의미는 아니다.**



## 정리

- 어떤 함수에 들어가는 모든 배개변수에 대해 타입 변환을 해줄 필요가 있다면, 그 함수는 비멤버 함수여야 한다.

