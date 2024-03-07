# ==와 .equals()

## 💡 **==**

==와 ≠ 연산자는 기본 자료형에서만 값 비교를 위해 사용할 수 있다.

참조 자료형에서 사용해도 되지만 **참조 자료형에서 사용하면** 값을 비교하는 게 아니라 **주소값**을 비교한다.

```java
Parent parent1 = new Parent("hyun");
Parent parent2 = new Parent("hyun");

System.out.println(parent1 == parent2); //false
```

Parent 객체 안에 `String name` 이라는 변수에 같은 값을 넣고 비교해도 결과는 false다.

두 객체는 각자의 생성자를 사용하여 만들었기 때문에 주소값이 다르기 때문이다.

<br> 

## 💡 **equals()**

참조 자료형은 equals()를 사용하여 비교해야 한다.

equals()를 오버라이딩 하지 않으면 hashCode값을 비교하므로 오버라이딩해서 비교해줘야 한다.

**오버라이딩할 때는 hashCode()도 같이 오버라이딩 해야 한다.**

만약 어떤 두 개의 객체가 서로 동일하다면, hashCode() 값은 무조건 동일해야만 한다.

왜냐면 equals()메서드를 오버라이딩해서 객체가 서로 같다고는 할 수 있지만 그 값이 같다고 해서 그 객체의 주소 값이 같지는 않기 때문이다.

→ equals()결과가 true여도 hashCode() 값은 서로 다르기 때문이다.

```java
//기본 equals() 
public boolean equals(Object obj) {
    return (this == obj);
}
```

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Parent parent = (Parent) o;
    return Objects.equals(name, parent.name);
}

@Override
public int hashCode() {
    return Objects.hash(name);
}
```

<br> 

### **매번 해야 하는가?**

DTO를 만들 경우에는 객체 비교를 위해서 반드시 필요하지만,

메소드만 있는 기능 위주의 클래스를 만들때는 할 필요 없다.