---
tags:
  - fragment
---
# 기본적인 개념 정리
[[!Resources/Excalidraw/Shared Resource|Shared Resource]]
**프로세스(Process)** : 운영체제에서 실행중인 프로그램의 **작업단위** 
좀더 쉽게 말하면 프로그램이 실제로
실행되고 있는 모습을 프로세스라고 할 수 있다.
![[!Resources/Excalidraw/Shared Resource#^group=iBwCvRCgnJnGnoxhW5SBb|700]]
그리고 일반적으로 한 프로세스에 반드시 **한개 이상의 스레드가** 있다. ^26cc31

**스레드(Thread)** : 프로세스 보다 **더 작은 작업단위**
프로세스가 작업을 수행하는 동안, 스레드는 그 작업 내에서 여러 하위 작업을 맡음
![[!Resources/Excalidraw/Shared Resource#^group=ZSb2iV_jZbQW0NuT3hgJa||700]]

프로세스 하나에 여러개의 스레드가 존재할 수 있다.
이를 **멀티스레딩 (Multi-threading)** 이라고 한다.

여러 하위작업들 동시에 처리하고, 스레드끼리 **서로 자원을 공유**하기 때문에 작업의 효율이 굉장히 올라간다. 
스레드가 함께 접근할 수 있는 메모리, 데이터 등의 자원을 **공유자원(Shared resource)** 라고 한다.
![[!Resources/Excalidraw/Shared Resource#^group=54FspADZ5RXyx6MtGP5lK|700]]

# 공유자원 & 임계영역
그러나 멀티스레딩은 문제점이 있는데, 각 스레드의 모든 흐름이 **동시에**, **각자** 작동한다는 점이다.
 한정된 공유자원을 여러 스레드에서 동시에 사용하다보니 
 하나의 자원을 두개 이상의 스레드가 동시에 읽거나 쓰는 상황도 일어나게 되었다.
이러한 상태를 **경쟁상태 (race condition)** 라고 한다.

심지어 프로세스도 하나만 있는것이 아니라 여러가지 프로세스를 동시에 실행할 수 있다. 
이를 **멀티프로세싱** 또는 **멀티테스킹**이라고 한다.
![[!Resources/Excalidraw/Shared Resource#^group=_EoZpsdeKYXsJjLUan9Sv|600]]


이렇게 복잡하게 멀티테스킹과 멀티스레딩이 이루어지는 상황에서 
경쟁상태인 두 스레드가 하나의 자원에 접근했을때
![[CleanShot 2024-08-08 at 17.30.59@2x.jpg|625]]
작업의 타이밍이 꼬여버리면 예기치 못한 결과를 초래할 수도 있다.
이러한 문제가 일어나는 공유자원을 **임계 영역(critical section)** 이라고 한다.

## 임계영역의 해결 모델

우선 가장 기본적인 해결방법의 매커니즘은 **잠금 (Lock)** 이다.
어떠한 공유자원을 먼저 사용한 스레드나 프로세스가 Lock을 걸어서 독점한 후, 처리가 끝나면 Unlock으로 다음 프로세스가 접근가능하게 만드는 방식이다.
이 Lock을 사용하는 방식은 크게 3가지가 있다.
#### 뮤텍스 (mutex)
가장 간단하고 직관적인 동기화 도구
한번에 하나의 스레드만 공유자원에 접근하고, 사용한 후에 다음 스레드가 공유자원에 접근하는 방식
![[!Resources/Excalidraw/Shared Resource#^group=_4ihUwbtd9Pyn-Nofle_n|700]]

뮤텍스 방식을 자바코드로 구현하면 요런 느낌인데,
~~~java
while (true){
	// Lock이 1이 될때 까지 대기
	aquireLock()
	 
	if(Lock = 1){
		// Lock 잠금 (Lock = 0)
		lock()
		...
		// Unlock (Lock = 1)
		Unlock()
		break;
	}
}						
~~~
Lock을 얻을때 까지 무한대로 While문을 돌린다. 이를 **바쁜 대기 (Busy Waiting)** 이라 하고,
CPU의 성능을 낭비하게 된다.
또한 모든 스레드가 직접 열고 잠그는 로직을 가지고 있어야 하기 때문에 코딩과 디버깅이 비효율적

### 세마포어 (semaphore)
공유자원을 분할해서 더 많은 스레드가 동시에 이용할 수 있도록 함.
`lock()` - `unlock()` 로직대신에`wait()` - `signal()` 로직을 사용한다. 
0과 1 만 존재하던 뮤텍스와 달리 세마포어는 양의 정수 값을 갖는다.
![[!Resources/Excalidraw/Shared Resource#^group=uhJXRshM3rt7RrTN0ngNf|700]]

화장실로 비유하자면
- 뮤텍스 : 한명이 볼일을 볼때 화장실 전체를 잠금 - 앞사람에게 화장실 열쇠를 받아야 뒷사람이 화장실 사용가능
- 세마포어 : 한명에게 화장실칸 한칸씩 부여 

### 모니터
**인터페이스와 큐(queue) 활용**
공유자원과 스레드의 연결이 직접적으로 일어나지 않게 막는다.
흡사 Spring에서의 IoC처럼 직접적인 의존관계를 만들지 않는것과 비슷한 느낌이다. [[0. 오브젝트와 의존관계#인터페이스 도입|IOC Interface]]
![[!Resources/Excalidraw/Shared Resource#^group=3-RPSlYL-ua7tmsrMcMZb|700]]

동시성을 처리하는 방법에 있어서 완성형이라고 할 수 있다.
타 스레드에게 자원을 넘겨받기 때문에 Lock같은 개념이 없어도 자동으로 상호배제가 된다.
어느정도 병목현상이 있을 수 있지만, 앞선 두 모델에 비해서 훨씬 효율적이고 안정적이다

---
# 교착상태
Deadlock
![[!Resources/Excalidraw/Shared Resource#^group=gCh-jmTq2kyz9vVtonOJa|700]]
말 그대로 서로 무한히 공유자원을 기다리며 중단된 상태
A는 B가 가진 자원을 요청, B는 A가 가진 자원을 요청
둘다 Lock을 풀지 않음

### 교착상태의 원인
4가지 조건이 모두 충족되어야 한다.
- 환형대기 : 서로의 자원을 요구하는 상태
- 상호배제 : 한 프로세스가 자원을 독점 해야 한다. (동시 사용을 막아야 하기 때문)
- 점유 대기 : 없는 자원을 사용하려면 대기해야 한다.
- 비선점 : 다른 프로세스의 자원을 강제로 뺏을 수 없다.

### 교착 상태의 해결 방법
자원할당 모델을 잘 설계하는것이 최선이다.
1. 예방 : 교착 상태의 4조건을 위반하여 교착상태가 발생하지 않도록  (다른 문제 발생 가능성 높음)
2. 회피 :시스템을 모니터링하여 안정상태를 유지하면서 자원을 할당 (시스템이 느려짐)

여러 방법론이 있지만, 이를 처리하는 비용이 더 커서 보통은 사용자가 컴퓨터를 껏다 키는게 최고의 방법이라고 한다...

