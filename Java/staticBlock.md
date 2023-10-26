# static 블록

## 💡언제 사용하는가?

객체는 여러 개를 생성하지만, 한 번만 호출되어야 하는 코드가 있다면 ?

static 블록을 사용하면 된다.

```java
static {
	//딱 한번만 수행되는 코드
}
```

이 static 블록은 객체가 생성되기 전에 한 번만 호출되고, 그 이후에는 호출하려고 해도 호출할 수가 없다.

## 💡예제

```java
public class StaticBlock {
		static int data = 1;
		public StaticBlock() {
			System.out.println("StaticBlock Constructor.");
		}

		static {
			System.out.println("** first static block **");
			data = 3;
		}

		static {
			System.out.println("** second static block **");
			data = 5;
		}

		public static int getData() {
			return data;
		}
}
```

```java
public class StaticBlockCheck {
    public static void main(String[] args) {
        StaticBlockCheck check = new StaticBlockCheck();
        check.makeStaticBlockObject();
    }

    private void makeStaticBlockObject() {
        System.out.println("Creating block1");
        StaticBlock block1 = new StaticBlock();
        System.out.println("Created block1");
        System.out.println("---------------");
        System.out.println("Creating block2");
        StaticBlock block2 = new StaticBlock();
        System.out.println("Created block2");
    }
}
```

### 실행 결과

```
Creating block1
** first static block **
** second static block **
StaticBlock Constructor.
Created block1
---------------
Creating block2
StaticBlock Constructor.
Created block2
```

- 선언된 순서대로 호출된다.
- 두 개의 `StaticBlock`객체를 만들었지만, static 블록들은 한 번씩만 호출되었다.
- 생성자가 호출되기 전에 static 블록들이 호출된다.
- static 블록 안에서는 static한 것들만 호출할 수 있다.
    - 만약 `StaticBlock`클래스의 `data` 가 static이 아니었다면 컴파일이 되지 않는다.
- 클래스 내에 선언되어 있어야 한다. 메소드 내에서는 선언할 수 없다.
    - 즉, 인스턴스 변수나 클래스 변수와 같이 어떤 메소드나 생성자에 속해 있으면 안 된다.

## 💡예제2  - 변수 값 출력
```java
public void makeStaticBlockObjectWithData() {
    System.out.println("data=" + StaticBlock.getData());
    StaticBlock block1 = new StaticBlock();
    block1.data = 1;
    System.out.println("------------------");
    StaticBlock block2 = new StaticBlock();
    block2.data = 2;
    System.out.println("data=" + StaticBlock.getData());
}
```

### 실행 결과

```
** first static block **
** second static block **
data=5
StaticBlock Constructor.
------------------
StaticBlock Constructor.
data=2
```

- data 값 출력 전에 static 블록이 호출되었다.

→ static 블록은 생성자가 불리지 않더라도 클래스에 대한 참조가 발생하자마자 호출된다.

- 두 번째 static 블록에 의해 `data = 5` 가 출력되었다.