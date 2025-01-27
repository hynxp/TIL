템플릿 메서드(Template Method) 패턴은 **여러 클래스에서 공통으로 사용하는 메서드를 템플릿화** 하여 상위 클래스에서 정의하고, **하위 클래스마다 세부 동작 사항을 다르게 구현**하는 패턴이다.

즉, **변하지 않는 기능(템플릿)은 상위 클래스에** 만들어두고 **자주 변경되며 확장할 기능은 하위 클래스에서** 만들도록 하여, 상위의 메소드 실행 동작 순서는 고정하면서 세부 실행 내용은 다양화 될 수 있는 경우에 사용된다.

템플릿 메소드 패턴은 상속이라는 기술을 극대화하여, **알고리즘의 뼈대**를 맞추는 것에 초점을 둔다. 이미 수많은 프레임워크에서 많은 부분에 템플릿 메소드 패턴 코드가 우리도 모르게 적용되어 있다.


## 주요 개념
1. **템플릿 메서드**: 알고리즘의 전체 구조를 정의하는 메서드로, 상위 클래스에 구현된다. 일부 단계는 하위 클래스에서 구현하도록 추상 메서드로 선언된다.
2. **훅 메서드(Hook Method)**: 하위 클래스에서 선택적으로 오버라이드할 수 있는 메서드로, 필수 구현 사항은 아니다.

## 구조
![[Pasted image 20250126091509.png|500]]
1. **AbstractClass**: 템플릿 메서드를 정의하고, 변해야 할 부분을 추상 메서드로 선언한다.
2. **ConcreteClass**: AbstractClass를 상속받아 추상 메서드를 구현하고, 템플릿 메서드에서 호출되는 동작을 정의한다.

### 훅(hook) 메서드
![[Pasted image 20250126091611.png|300]]
훅(hook) 메소드는 부모의 템플릿 메서드의 **영향이나 순서를 제어**하고 싶을때 사용되는 메서드 형태를 말한다.

위의 그림에서 보듯이 템플릿 메서드 내에 실행되는 동작을 ~~step2()~~ 이라는 메서드의 참, 거짓 유무에 따라 다음 스텝을 어떻게 이어나갈지 지정한다. 이를 통해 자식 클래스에서 좀더 유연하게 템플릿 메서드의 알고리즘 로직을 다양화 할 수 있다는 특징이 있다.

훅 메소드는 추상 메소드가 아닌 일반 메소드로 구현하는데, 선택적으로 오버라이드 하여 자식 클래스에서 제어하거나 아니면 놔두거나 하기 위해서 이다.


## 예제 - 커피와 차 만드는 과정
커피와 차를 만드는 과정은 대부분 비슷하지만, 특정 부분(재료 추가)은 다르다. 이를 템플릿 메서드 패턴으로 구현할 수 있다.

### AbstractClass: 음료 만들기
```java
public abstract class Beverage {
    // 템플릿 메서드
    public final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }

    private void boilWater() {
        System.out.println("물을 끓인다");
    }

    private void pourInCup() {
        System.out.println("컵에 따른다");
    }

    // 추상 메서드
    protected abstract void brew(); // 각 음료의 추출 방법
    protected abstract void addCondiments(); // 첨가물 추가
}
```

### ConcreteClass: 커피
```java
public class Coffee extends Beverage {
    @Override
    protected void brew() {
        System.out.println("커피를 추출한다");
    }

    @Override
    protected void addCondiments() {
        System.out.println("설탕과 우유를 추가한다");
    }
}
```

### ConcreteClass: 차
```java
public class Tea extends Beverage {
    @Override
    protected void brew() {
        System.out.println("차를 우린다");
    }

    @Override
    protected void addCondiments() {
        System.out.println("레몬을 추가한다");
    }
}
```

### 클라이언트 코드
```java
public class Main {
    public static void main(String[] args) {
        Beverage coffee = new Coffee();
        coffee.prepareRecipe();
        // 출력:
        // 물을 끓인다
        // 커피를 추출한다
        // 컵에 따른다
        // 설탕과 우유를 추가한다

        Beverage tea = new Tea();
        tea.prepareRecipe();
        // 출력:
        // 물을 끓인다
        // 차를 우린다
        // 컵에 따른다
        // 레몬을 추가한다
    }
}
```

## 템플릿 메서드 패턴의 장점
1. **코드 재사용성**: 알고리즘의 공통 부분을 상위 클래스에서 정의하여 중복 코드를 제거할 수 있다.
2. **유지보수성**: 알고리즘의 전체 구조가 상위 클래스에 집중되어 있어 변경이 용이하다.
3. **확장성**: 하위 클래스에서 세부 구현을 변경하거나 추가하여 쉽게 확장할 수 있다.

## 템플릿 메서드 패턴의 단점
1. **상속의 의존성**: 상속을 기반으로 하기 때문에, 지나치게 많은 하위 클래스가 생길 수 있다.
2. **구조적 제약**: 상위 클래스와 하위 클래스 간의 강한 결합이 생길 수 있다.


## 요약
템플릿 메서드 패턴은 알고리즘의 뼈대를 상위 클래스에서 정의하고, 하위 클래스에서 변해야 할 부분만 구현하도록 설계하는 강력한 패턴이다. 이는 코드의 재사용성과 일관성을 유지하면서도 다양한 상황에 맞는 확장을 가능하게 한다. 
하지만 상속 기반의 설계로 인해 지나친 의존성이 생기지 않도록 주의해야 한다.



참고
[템플릿 메소드(Template Method) 패턴 - 완벽 마스터하기](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A9%94%EC%86%8C%EB%93%9CTemplate-Method-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)
