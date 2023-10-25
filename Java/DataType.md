# 기본 자료형과 참조 자료형

> new를 사용해서 초기화하는 것을 **참조 자료형**
> <br>
>그렇지 않고 바로 초기화가 가능한 것을 **기본 자료형**이라고 한다.

딱 하나 예외가 있다.
바로 **String**!

String은 두 가지 방법으로 초기화 가능하다.

- String str = “String”;
- String str = new String(”String”);

기본 자료형

1. 숫자
    1. 정수형 : byte, short, int, long, char
    2. 소수형 : float, double
2. boolean

![image](https://github.com/kyunghyun-Park/TIL/assets/50633008/29d13569-2091-4647-a6de-e92c6cea486d)

### 💡byte 연산
***

```java
byte byteMin = -128;
byte byteMax = 127;
System.out.println(byteMin-1); //127
System.out.println(byteMax+1); //-128
```

최소값에서 1을 더 뺀 것은 최대값이 나오고,

최대값에서 1을 더한 것은 최소값이 나온다.

왜일까?

byteMin을 2진수로 표현하면 1000_0000인데 여기서 -1 하면 0111_1111

byteMax값 0111_1111에서 1을 더하면 1000_0000이 되기 때문이다.

### 💡변수의 초기화
***

자바의 모든 자료형은 값을 지정하지 않으면 기본값을 사용한다.

하지만 지역 변수로 기본 자료형을 사용할 때에는 반드시 값을 지정해야만 한다.

값을 지정하지 않고 개발하는 것은 매우 안좋은 습관이므로 **명시적으로 기본값을 지정**하자.