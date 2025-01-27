프록시 패턴은 **다른 객체에 대한 대리자(Proxy)를 제공하여, 그 객체에 대한 접근을 제어하거나 추가적인 기능을 제공하는 디자인 패턴**이다. 주로 객체에 직접 접근하기 어려운 상황에서 이를 대신 수행하거나, 객체 사용 전후에 부가적인 작업을 수행하고 싶을 때 사용된다.

## 프록시 패턴의 구조
프록시 패턴은 세 가지 주요 구성 요소로 이루어진다

1. **Subject (주체)**: 프록시와 실제 객체가 구현하는 공통 인터페이스.
2. **RealSubject (실제 객체)**: 실제 작업을 수행하는 객체.
3. **Proxy (대리자)**: `RealSubject`에 대한 대리 객체로, 요청을 전달하거나 추가 기능을 제공한다.

### 클래스 다이어그램
![[Pasted image 20250126080614.png]]
#### Subject : Proxy와 RealSubject를 하나로 묶는 인터페이스 (다형성)
- 대상 객체와 프록시 역할을 동일하게 하는 추상 메소드 ~~operation()~~ 를 정의한다.
- 인터페이스가 있기 때문에 클라이언트는 Proxy 역할과 RealSubject 역할의 차이를 의식할 필요가 없다.
#### RealSubject : 원본 대상 객체
#### Proxy : 대상 객체(RealSubject)를 중계할 대리자 역할
- 프록시는 대상 객체를 합성(composition)한다.
- 프록시는 대상 객체와 같은 이름의 메서드를 호출하며, 별도의 로직을 수행 할수 있다 (인터페이스 구현 메소드)
- 프록시는 흐름제어만 할 뿐 결과값을 조작하거나 변경시키면 안 된다.
#### Client : Subject 인터페이스를 이용하여 프록시 객체를 생성해 이용.
- 클라이언트는 프록시를 중간에 두고 프록시를 통해서 RealSubject와 데이터를 주고 받는다.


## 활용 사례
프록시 패턴은 다양한 시나리오에서 유용하다

### 보안(Security)
프록시는 클라이언트가 작업을 수행할 수 있는 권한이 있는지 확인하고 검사 결과가 긍정적인 경우에만 요청을 대상으로 전달한다.

### 캐싱(Caching
프록시가 내부 캐시를 유지하여 데이터가 캐시에 아직 존재하지 않는 경우에만 대상에서 작업이 실행되도록 한다.

### 데이터 유효성 검사(Data validation)
프록시가 입력을 대상으로 전달하기 전에 유효성을 검사한다.

### 지연 초기화(Lazy initialization)
대상의 생성 비용이 비싸다면 프록시는 그것을 필요로 할때까지 연기할 수 있다.

### 로깅(Logging)
프록시는 메소드 호출과 상대 매개 변수를 인터셉트하고 이를 기록한다.

### 원격 객체(Remote objects)
프록시는 원격 위치에 있는 객체를 가져와서 로컬처럼 보이게 할 수 있다.


## 예제
### 이미지 로딩을 위한 가상 프록시
이미지 로딩은 시간이 오래 걸릴 수 있으므로, 실제 이미지를 필요할 때만 로드하는 프록시를 구현할 수 있다.

#### Subject 인터페이스
```java
public interface Image {
    void display();
}
```

#### RealSubject (실제 이미지 객체)
```java
public class RealImage implements Image {
    private String fileName;

    public RealImage(String fileName) {
        this.fileName = fileName;
        loadImageFromDisk();
    }

    private void loadImageFromDisk() {
        System.out.println("Loading " + fileName);
    }

    @Override
    public void display() {
        System.out.println("Displaying " + fileName);
    }
}
```

#### Proxy (대리 객체)
```java
public class ProxyImage implements Image {
    private String fileName;
    private RealImage realImage;

    public ProxyImage(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName); // 필요할 때만 RealImage 생성
        }
        realImage.display();
    }
}
```

#### 클라이언트 코드
```java
public class Main {
    public static void main(String[] args) {
        Image image = new ProxyImage("test.jpg");

        // 실제 이미지를 처음 로드할 때만 파일을 읽음
        image.display(); // 출력: Loading test.jpg, Displaying test.jpg

        // 이미 로드된 이미지는 캐싱
        image.display(); // 출력: Displaying test.jpg
    }
}
```

## 장점
#### [개방 폐쇄 원칙(OCP)Visit Website](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-OCP-%EA%B0%9C%EB%B0%A9-%ED%8F%90%EC%87%84-%EC%9B%90%EC%B9%99) 준수
기존 대상 객체의 코드를 변경하지 않고 새로운 기능을 추가할 수 있다.

#### [단일 책임 원칙(SRP)Visit Website](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-SRP-%EB%8B%A8%EC%9D%BC-%EC%B1%85%EC%9E%84-%EC%9B%90%EC%B9%99) 준수 
대상 객체는 자신의 기능에만 집중 하고, 그 이외 부가 기능을 제공하는 역할을 프록시 객체에 위임하여 다중 책임을 회피 할 수 있다.

#### 원래 하려던 기능을 수행하며 그외의 부가적인 작업(로깅, 인증, 네트워크 통신 등)을 수행하는데 유용하다
#### 클라이언트는 객체를 신경쓰지 않고, 서비스 객체를 제어하거나 생명 주기를 관리할 수 있다.
#### 사용자 입장에서는 프록시 객체나 실제 객체나 사용법은 유사하므로 사용성에 문제 되지 않는다.


## 단점
### 복잡도 증가
많은 프록시 클래스를 도입해야 하므로 코드의 복잡도가 증가한다.  
예를들어 여러 클래스에 로깅 기능을 가미 시키고 싶다면, 동일한 코드를 적용함에도 각각의 클래스에 해당되는 프록시 클래스를 만들어서 적용해야 되기 때문에 코드량이 많아지고 중복이 발생 된다.

### 성능 저하
프록시 클래스 자체에 들어가는 자원이 많다면 서비스로부터의 응답이 늦어질 수 있다.



참고
[프록시(Proxy) 패턴 - 완벽 마스터하기](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%94%84%EB%A1%9D%EC%8B%9CProxy-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)