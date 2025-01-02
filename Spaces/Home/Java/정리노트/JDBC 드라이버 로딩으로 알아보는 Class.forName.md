```java
Class.forName("com.mysql.cj.jdbc.Driver");

// 데이터베이스 연결 
Connection connection = DriverManager.getConnection( "jdbc:mysql://localhost:3306/mydatabase", "root", "password" );
```
[[14. 데이터베이스(JDBC, 커넥션 풀)#JDBC|JDBC ]]드라이버를 로딩할 때 `Class.forName` 이라는 메소드를 사용하는데 이 메소드는 어떤 역할을 하는 메소드일까?
Java Reflection에서 제공하는 기능 중에 하나기도 하고, 클래스 로더랑도 관련이 있으니 알아보자.


## Class.forName()란?
`Class.forName` 메서드는 Java에서 클래스 이름을 문자열로 제공하여 해당 클래스를 로드하고 초기화하는 역할을 한다. 


## JDBC 드라이버 로딩 과정

### 1. 드라이버 클래스 이름 전달
```java
Class.forName("com.mysql.cj.jdbc.Driver");
```


### 2. 클래스 로드
`Class.forName`의 내부 코드를 보자.
```java
public static Class<?> forName(String className) throws ClassNotFoundException {  
    Class<?> caller = Reflection.getCallerClass();  
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);  
}
```
먼저 호출한 클래스가 뭔지 알아내기 위해 `Reflection.getCallerClass()`로 정보를 가져온다.

그리고 `forName`의 실제 작업을 수행하는 네이티브 메서드가 내부적으로 호출된다.

```java
@CallerSensitive  
public static Class<?> forName(String name, boolean initialize,  
                               ClassLoader loader)  
    throws ClassNotFoundException  
{  
    Class<?> caller = null;  
    
    @SuppressWarnings("removal")  
    SecurityManager sm = System.getSecurityManager();  
    
    if (sm != null) {  
        // Reflective call to get caller class is only needed if a security manager  
        // is present.  Avoid the overhead of making this call otherwise.        caller = Reflection.getCallerClass();  
        if (loader == null) {  
            ClassLoader ccl = ClassLoader.getClassLoader(caller);  
            if (ccl != null) {  
                sm.checkPermission(  
                    SecurityConstants.GET_CLASSLOADER_PERMISSION);  
            }  
        }  
    }  
    return forName0(name, initialize, loader, caller);  
}
```
위 코드를 자세히는 모르겠지만 우선 **호출한 클래스의 클래스 로더를 가져온다.**
특정 클래스 로더를 명시하지 않으면 기본적으로 호출한 클래스와 동일한 클래스 로더를 사용한다.
JDBC 드라이버 로딩의 경우, 드라이버 클래스가 포함된 JAR 파일이 이 클래스 로더의 클래스패스에 있어야 로드가 성공한다.

### Driver 정적 블록 실행
드라이버 클래스가 로드되면 해당 클래스의 정적 블록이 실행된다.

이 정적 블록에서 드라이버가 `DriverManager`에 등록된다.
```java
public class Driver implements java.sql.Driver {
    static {
        try {
            // 드라이버 등록
            DriverManager.registerDriver(new Driver());
        } catch (SQLException e) {
            throw new RuntimeException("Failed to register driver", e);
        }
    }

    public Driver() {
        // 생성자
    }
}
```
static절(정적 블록)은 클래스가 로딩되는 시점에 초기화를 위해 실행되는 부분이다.
대부분의 JDBC 드라이버는 이 정적 블록에서 자신을 `DriverManager`에 등록한다.

이렇게 static 블록을 가지는 클래스들은 Class.forName을 호출하기만 해도 초기화가 실행되기 때문에, 인스턴스를 별도로 관리하지 않는 클래스에서 자주 활용된다고 한다. 이렇게 static 블록에서 클래스 자기 자신의 인스턴스를 생성하고 관리합니다.

```java
public static void registerDriver(java.sql.Driver driver, DriverAction da) throws SQLException {  
  
    /* Register the driver if it has not already been added to our list */  
    if (driver != null) {  
        registeredDrivers.addIfAbsent(new DriverInfo(driver, da));  
    } else {  
        // This is for compatibility with the original DriverManager  
        throw new NullPointerException();  
    }  
  
    println("registerDriver: " + driver);
}
```
`registerDriver()`은 현재 DriverManager에 등록된 드라이버가 없을 때에만 드라이버를 등록해주는 메소드이다.  드라이버가 이미 등록되어 있다면 중복 등록을 방지해준다.
Class.forName을 여러 번 호출하더라도 드라이버 객체가 여러 번 생성되지 않도록 설계된 것이다.


이로써 forName의 역할이 단지 Class 객체를 반환하는 것이 아니라 클래스 로더에 의해 그 클래스 객체가 로드된다는 것을 알 수 있다.








참고
[JDBC에서 Class.forName과 클래스 로딩에 대해 알아보기](https://limdevbasic.tistory.com/26)