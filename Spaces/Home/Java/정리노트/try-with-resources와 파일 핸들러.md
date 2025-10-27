파일을 다루는 프로그램을 작성하다 보면 "파일을 닫아야 한다"는 말을 자주 듣게 된다. 
하지만 왜 파일을 닫아야 하는지, 닫지 않으면 어떤 문제가 발생하는지 정확히 이해하기는 쉽지 않다. 이 글에서는 파일 핸들의 개념부터 시작해서, try-with-resources를 사용해야 하는 이유까지 차근차근 알아보겠다.

## 파일 핸들이란 무엇인가
파일 핸들(File Descriptor)을 이해하기 위해 영화관 비유를 들어보자. 영화관에서 영화를 보려면 매표소에서 표를 받아야 한다. 이 표에는 좌석 번호가 적혀 있고, 우리는 그 번호를 보고 자리를 찾아간다. 영화가 끝나면 자리에서 일어나면 되는데, 만약 계속 자리에 앉아 있으면 그 좌석은 다른 사람이 사용할 수 없게 된다.

여기서 영화관은 운영체제(OS), 좌석은 파일, 좌석 번호표는 파일 핸들에 해당한다. 파일 핸들은 운영체제가 열린 파일을 관리하기 위해 부여하는 숫자 식별자다. 프로그램이 파일을 열면 운영체제는 0, 1, 2, 3과 같은 정수 번호를 할당하고, 이 번호를 통해 파일을 읽거나 쓸 수 있게 된다.


## 파일을 열 때 내부에서 일어나는 일
자바에서 파일을 열 때 내부적으로 어떤 일이 일어나는지 살펴보자.

```java
BufferedReader br = new BufferedReader(new FileReader("data.csv"));
```

이 코드를 실행하면 다음과 같은 과정이 진행된다. 먼저 자바 프로그램이 운영체제에게 "data.csv 파일을 열어달라"고 요청한다. 운영체제는 파일 시스템을 확인해서 해당 파일이 존재하는지 검사한다. 파일이 존재하면 파일 핸들 테이블에서 사용 가능한 빈 번호를 찾는다. 예를 들어 3번이 비어있다면 FD 3번을 할당하고, 이 번호를 자바 프로그램에 반환한다. BufferedReader 객체는 내부적으로 이 번호를 저장하고 있다가, 이후 파일을 읽거나 쓸 때 이 번호를 사용한다.


## 파일 핸들 테이블의 실제 모습
모든 프로세스는 자신만의 파일 핸들 테이블을 가지고 있다. 이 테이블은 다음과 같은 형태로 구성된다.

|FD|파일 경로|상태|
|---|---|---|
|0|/dev/stdin|열림 (표준 입력)|
|1|/dev/stdout|열림 (표준 출력)|
|2|/dev/stderr|열림 (표준 에러)|
|3|/home/user/data.csv|열림|
|4|(비어있음)|-|
|5|(비어있음)|-|
|...|...|...|
|1023|(비어있음)|-|

여기서 주목할 점은 0, 1, 2번은 이미 표준 입출력으로 예약되어 있다는 것이다. 따라서 우리가 처음 파일을 열면 보통 3번부터 할당받게 된다.

## 파일을 읽을 때의 동작 과정

파일 핸들을 할당받은 후 파일을 읽는 과정을 살펴보자.

```java
String line = br.readLine();
```

이 코드가 실행되면 자바 프로그램은 운영체제에게 "3번 파일에서 한 줄을 읽어달라"고 요청한다. 운영체제는 FD 3번에 해당하는 파일 경로(/home/user/data.csv)를 테이블에서 찾아서, 실제 파일 시스템에서 데이터를 읽어온다. 그리고 읽은 데이터를 자바 프로그램에 반환하면, 프로그램은 이 데이터를 line 변수에 저장한다.

## close를 하지 않으면 발생하는 문제
파일을 사용한 후 close()를 호출하지 않으면 심각한 문제가 발생할 수 있다.

```java
// 파일 1 열기
BufferedReader br1 = new BufferedReader(new FileReader("file1.csv"));  // FD 3
// close() 안 함

// 파일 2 열기
BufferedReader br2 = new BufferedReader(new FileReader("file2.csv"));  // FD 4
// close() 안 함

// 파일 3 열기
BufferedReader br3 = new BufferedReader(new FileReader("file3.csv"));  // FD 5
// close() 안 함
```

이런 식으로 파일을 계속 열기만 하고 닫지 않으면, 파일 핸들 테이블이 다음과 같이 채워진다.

|FD|파일|상태|
|---|---|---|
|0|stdin|열림|
|1|stdout|열림|
|2|stderr|열림|
|3|file1.csv|열림 (안 닫음)|
|4|file2.csv|열림 (안 닫음)|
|5|file3.csv|열림 (안 닫음)|
|...|...|...|
|1023|file1021.csv|열림 (안 닫음)|
|1024|❌ 한계 초과|"Too many open files"|

프로세스당 할당할 수 있는 파일 핸들의 개수는 제한되어 있다. 일반적으로 1024개 정도의 제한이 있는데, 이 한계에 도달하면 "Too many open files"라는 에러가 발생하고 더 이상 파일을 열 수 없게 된다.

## close를 호출하면 일어나는 일
반대로 제대로 close()를 호출하면 어떻게 될까?

```java
BufferedReader br = new BufferedReader(new FileReader("data.csv"));  // FD 3 할당
String line = br.readLine();
br.close();  // FD 3 반납
```

close()를 호출하면 할당받았던 파일 핸들이 운영체제에 반납된다. 파일 핸들 테이블에서 해당 번호가 다시 "비어있음" 상태가 되어, 다른 파일을 열 때 재사용할 수 있게 된다.

|상태|FD 3|
|---|---|
|close() 전|data.csv (사용 중)|
|close() 후|(비어있음)|

## 실제로 확인해보기
리눅스나 맥에서는 다음 명령어로 현재 프로세스가 열고 있는 파일 목록을 확인할 수 있다.

```bash
lsof -p [프로세스ID]
```

또는 다음과 같이 확인할 수도 있다.

```bash
ls -la /proc/[프로세스ID]/fd
```

출력 결과는 다음과 같은 형태로 나타난다.

|FD|파일 경로|
|---|---|
|0|/dev/pts/0 (터미널 입력)|
|1|/dev/pts/0 (터미널 출력)|
|2|/dev/pts/0 (에러 출력)|
|3|/home/user/data.csv|
|4|/var/log/app.log|

자바 코드에서도 현재 열린 파일 핸들의 개수를 확인할 수 있다.

```java
import java.io.*;
import java.lang.management.*;
import com.sun.management.UnixOperatingSystemMXBean;

public class FileDescriptorDemo {
    public static void main(String[] args) throws Exception {
        OperatingSystemMXBean os = ManagementFactory.getOperatingSystemMXBean();
        
        if (os instanceof UnixOperatingSystemMXBean) {
            UnixOperatingSystemMXBean unix = (UnixOperatingSystemMXBean) os;
            System.out.println("열린 파일 핸들 수: " + unix.getOpenFileDescriptorCount());
            System.out.println("최대 파일 핸들 수: " + unix.getMaxFileDescriptorCount());
        }
        
        // 파일 100개 열기 (close 안 함)
        for (int i = 0; i < 100; i++) {
            new FileReader("data.csv");  // close() 안 함
            
            if (os instanceof UnixOperatingSystemMXBean) {
                UnixOperatingSystemMXBean unix = (UnixOperatingSystemMXBean) os;
                System.out.println("현재 열린 파일: " + unix.getOpenFileDescriptorCount());
            }
        }
    }
}
```

이 코드를 실행하면 파일을 열 때마다 열린 파일 핸들의 개수가 증가하는 것을 확인할 수 있다. 출력 결과는 다음과 같다.

```
열린 파일 핸들 수: 3
최대 파일 핸들 수: 1024
현재 열린 파일: 4
현재 열린 파일: 5
현재 열린 파일: 6
...
현재 열린 파일: 103
```

## 파일 핸들 개념 정리
파일 핸들을 책 대출에 비유해보자. 
파일은 도서관에 있는 책이고, 파일 핸들은 대출증 번호다. 책을 빌리면 대출증을 받게 되고, 이 대출증으로 책을 읽을 수 있다. 책을 다 읽고 반납하면 대출증도 함께 반납하게 된다. 만약 책을 반납하지 않으면 도서관의 대출 한도가 초과되어 더 이상 책을 빌릴 수 없게 된다.

핵심 개념을 표로 정리하면 다음과 같다.

|용어|의미|예시|
|---|---|---|
|파일 핸들|OS가 부여한 파일 식별 번호|3, 4, 5, ...|
|할당|파일 열 때 번호 받음|open("file.csv") → FD 3|
|해제|파일 닫을 때 번호 반납|close() → FD 3 반납|
|테이블|프로세스별 FD 목록|FD 0~1023|
|한계|프로세스당 최대 개수|보통 1024개|

## try-with-resources를 사용해야 하는 이유
파일 핸들의 개념을 이해했다면, 이제 왜 try-with-resources를 사용해야 하는지 알아보자. 전통적인 방식으로 파일을 다루면 다음과 같이 코드를 작성해야 한다.

```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("data.csv"));
    String line = br.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (br != null) {
        try {
            br.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

이 코드의 문제점은 명확하다. finally 블록에서 명시적으로 close()를 호출해야 하고, close() 자체도 예외를 던질 수 있어서 또 다른 try-catch가 필요하다. 코드가 장황해지고, 실수로 close()를 빼먹을 가능성도 있다.

Java 7부터 도입된 try-with-resources를 사용하면 이런 문제를 깔끔하게 해결할 수 있다.

```java
try (BufferedReader br = new BufferedReader(new FileReader("data.csv"))) {
    String line = br.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
}
```

try 괄호 안에서 리소스를 선언하면, try 블록이 끝날 때 자동으로 close()가 호출된다. 예외가 발생하더라도 반드시 close()가 실행되므로, 파일 핸들 누수를 방지할 수 있다.


## try-with-resources의 동작 원리
try-with-resources가 어떻게 자동으로 close()를 호출하는지 살펴보자. 이 기능을 사용하려면 리소스가 AutoCloseable 인터페이스를 구현해야 한다.

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

BufferedReader, FileReader, FileWriter 등 대부분의 IO 관련 클래스들은 이미 AutoCloseable을 구현하고 있다. try-with-resources 구문은 컴파일 시점에 다음과 같은 코드로 변환된다.

```java
BufferedReader br = new BufferedReader(new FileReader("data.csv"));
try {
    String line = br.readLine();
    System.out.println(line);
} finally {
    if (br != null) {
        br.close();
    }
}
```

즉, 개발자가 명시적으로 작성하지 않아도 컴파일러가 자동으로 finally 블록과 close() 호출을 추가해준다.

## 여러 리소스를 동시에 사용하기
try-with-resources는 여러 리소스를 동시에 관리할 수도 있다.

```java
try (BufferedReader br = new BufferedReader(new FileReader("input.csv"));
     BufferedWriter bw = new BufferedWriter(new FileWriter("output.csv"))) {
    
    String line;
    while ((line = br.readLine()) != null) {
        bw.write(line.toUpperCase());
        bw.newLine();
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

이 경우 리소스는 선언의 역순으로 close()가 호출된다. 즉, BufferedWriter가 먼저 닫히고 그 다음에 BufferedReader가 닫힌다. 이는 리소스 간의 의존성을 고려한 설계다.

## 예외 처리의 우아함
try-with-resources의 또 다른 장점은 예외 처리가 더 우아하다는 것이다. 전통적인 방식에서는 try 블록과 finally 블록에서 각각 예외가 발생할 수 있는데, 이 경우 finally의 예외가 try의 예외를 덮어씌우는 문제가 있었다.

```java
// 전통적인 방식
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("data.csv"));
    String line = br.readLine();
    throw new RuntimeException("읽기 중 에러");  // 이 예외가 사라짐
} finally {
    br.close();  // 여기서도 예외 발생 시, 위 예외는 손실됨
}
```

반면 try-with-resources는 suppressed exception 메커니즘을 사용해서 모든 예외 정보를 보존한다.
```java
try (BufferedReader br = new BufferedReader(new FileReader("data.csv"))) {
    String line = br.readLine();
    throw new RuntimeException("읽기 중 에러");  // 주 예외
    // close()에서 예외 발생 시 suppressed exception으로 기록됨
} catch (Exception e) {
    e.printStackTrace();  // 주 예외와 suppressed 예외 모두 출력
    Throwable[] suppressed = e.getSuppressed();  // suppressed 예외 확인 가능
}
```


## 실무에서의 활용
실무에서 파일 처리 로직을 작성할 때는 항상 try-with-resources를 사용해야 한다. 특히 다음과 같은 상황에서 더욱 중요하다.

```java
public void processLargeFile(String filePath) {
    try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
        String line;
        int count = 0;
        
        while ((line = br.readLine()) != null) {
            // 수백만 줄을 처리하는 중
            processLine(line);
            count++;
            
            if (count % 100000 == 0) {
                System.out.println("처리된 줄 수: " + count);
            }
        }
    } catch (IOException e) {
        System.err.println("파일 처리 중 오류 발생: " + e.getMessage());
    }
    // 이 시점에 BufferedReader는 자동으로 닫혀 있음
}
```

대용량 파일을 처리하는 경우, 중간에 예외가 발생하더라도 try-with-resources가 파일을 확실하게 닫아주므로 파일 핸들 누수를 걱정할 필요가 없다.