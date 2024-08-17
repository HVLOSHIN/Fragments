---
tags:
  - fragment
---
# Stack

### Intro

![[Pasted image 20240718182805.png|175]] \
사전적 의미 : 쌓아 올린 더미
뷔페의 접시, 프링글스, 쌓아둔 책 등으로 비유된다.\
이러한 일상의 개념을 추상화 하여 자료구조로 만든것이다.

### 용어, 개념 정리

- 후입 선출(Last-in-First-out) : 가장 마지막에 삽입된 데이터가 가장 먼저 제거

- top : 스택에서 자료가 삽입, 제거가 이루어지는 곳 push-pop이 top에서 이루어진다. top을 기반으로(스택에 가장 마지막에 담긴 데이터) 스택의 크기를 유추할 수 있다.

- push : 자료를 삽입

- pop : 자료를 제거

### Java 코드 에서의 Stack 활용

Java 에서는 `java.util.Stack` 클래스를 통해 Stack 기능을 제공한다.

```java
import java.util.Stack;

public class Stack1Ex {
    public static void main(String[] args) {

        // Integer형 스택 선언
        Stack<Integer> stack = new Stack<>();

        // 값 추가 push()
        stack.push(1);
        stack.push(2);
        stack.push(3);

        System.out.println(stack);

        // 값 제거 pop()
        stack.pop();
        System.out.println(stack);
        
        // Stack의 Top 출력 
        System.out.println(stack.peek());
    }
}
```

```java
[1, 2, 3]
[1, 2]
2
```

- Stack을 사용할때는 **제네릭 클래스**를 기본 포맷으로 한다.
- Stack 클래스에서 제공하는 메서드들도 있다.

후입선출의 성질을 이용하여 역순문자열을 출력하게 만들수 도 있다.

```java
public class Stack2Ex {
    public static void main(String[] args) {

        // String형 스택 선언
        Stack<String> stack = new Stack<>();

        // 값 추가 push()
        stack.push("!");
        stack.push("O");
        stack.push("L");
        stack.push("L");
        stack.push("E");
        stack.push("H");

       while (!stack.isEmpty()) {
           System.out.print(stack.pop());
       }
```

```java
HELLO!
```

### Java 메모리 구조 에서의 스택

JVM(Java Virtual Machine)에서 메모리를 크게 3가지 영역으로 나누어 관리를 한다.

- 힙 영역 (Heap) : 생략
- 메서드 영역 (Static) : 생략
- 스택 영역 (Stack) : 메서드 호출시 스택 프레임을 생성. 메서드가 종료시 메모리에서 제거

```java
public class Stack3Ex {
    public static int a = 10;

    public static void main(String[] args) {
        
        int b = 5;
        int c = 7;
        int result = a + b + c;
    }
}
```

간단한 예시를 들어보면

1. a 는 static 영역에 저장된다. static은 일종의 상비 대기 영역이라고 생각하면 편하다.

2. Main 이 시작되면서 main 스택 프레임 생성 (args 는 무시하겠음)

3. b는 스택영역의 맨 아래 들어오게 된다

4. c는 b 위에 쌓이게 된다.

5. result 는 스택의 맨 위에 쌓이게 된다.

이를 통해 알 수 있는 것

- 자바는 스택 영역을 사용해서 메서드 호출과 지역변수(+ 매개변수)를 관리한다
- 메서드를 계속 호출하면 스택 프레임이 계속 쌓인다
- 지역변수(+ 매개변수)는 스택영역에서 관리한다
- 스택프레임이 종료되면 지역변수도 함께 제거된다
- 스택 프레임이 모두 제거되면 프로그램도 종료된다.

### Stack Overflow

스택의 크기를 고정시켜서 사용할 경우, 스택의 크기를 넘어선 데이터를 삽입할 경우,

스택 오버플로우가 발생하게 된다.

반대로 스택이 비어있을때, 스택을 꺼내려고 할 경우, 스택 언더플로우가 발생

### Stack 시간 복잡도

스택에서 데이터 삽입, 삭제, isEmpty(), peek() 연산은 모두 O(1) 의 시간복잡도를 갖는다.

→ 삽입, 삭제는 항상 top에서만 일어나기 떄문에 특정 index를 찾아야 할 필요가 없음.

탐색의 경우에는 들어있는 데이터를 한번 훑어주면 되기 때문에에 O(n)

# Queue

### Intro
![[Pasted image 20240718182736.png|475]]
사전적 의미 : 꼬리, 줄 에서 유래

대기열이 일상에서 사용되는 가장 좋은 예

### 용어, 개념 정리

- 선입 선출(First-in-First-out) 순서를 따른다 Last-in-Last-out 도 가능한 표현이다.

- Front : 큐에서 맨 앞 (가장 먼저들어온 데이터)

- Rear : 맨 뒤 (가장 마지막에 들어온 데이터)

- enqueue(add) : rear 에 새로운 데이터를 추가

- dequeue (remove): front 항목을 제거

### Java 코드에서의 Queue 활용

```java
import java.util.LinkedList;
import java.util.Queue;

public class Queue1Ex {
    public static void main(String[] args) {
        Queue<Integer> que = new LinkedList<>();

        // 큐
        for (int i = 1; i <= 10; i++) {
            que.add(i);
        }
        System.out.println(que);
        System.out.println("peek : " + que.peek());

        for (int i = 1; i <= 3; i++) {
            que.remove();
        }
        System.out.println(que);
        System.out.println("peek : " + que.peek());
    }
}
```

```java
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
peek : 1
[4, 5, 6, 7, 8, 9, 10]
peek : 4
```

- add() : enqueue 데이터 삽입 메서드

- remove() : dequeue 데이터 삭제 메서드

- peek() : Front (가장 앞에 있는 데이터)를 반환하는 메서드 만약 Queue가 비어있다면 null을 반환한다.

- offer() , element() 라는 메서드도 있지만,

  비어있을시 null을 반환하는 add()/ remove()와 같은 동작을 하고 add, remove와 달리 예외를 던지기 때문에 안정성이 떨어진다.

### Queue 시간 복잡도

큐에서 데이터 삽입, 삭제 연산은 모두 O(1)의 시간 복잡도를 갖는다.

→ 삽입, 삭제는 항상 rear, front에서만 일어나기 떄문에 특정 index를 찾을 필요가 없음.

탐색의 경우 O(n)