---
tags:
  - java
  - fragment
text: "[[Java]]"
---

#### 새로운 Switch 문
~~~java
switch(변수){
	case 1 -> ;
	case 2 -> ;
	default -> ;
}
~~~

#### 삼항연산자
~~~java
(조건) ? a : b
~~~
조건이 참이면  a
조건이 거짓이면 b

~~~java
 int age = 20;
 String status = (age >= 18) ? "성인" : "미성년자"; //참이므로 status = "성인"
 System.out.println(status) // 결과 : 성인
 ~~~

#### do-while 문
~~~java
do{

}while(조건);
~~~
한번은 동작을 수행하고 조건에 따라 반복.

#### break, continue

~~~java
While(조건식){
	코드1;
	break; // 즉시 while문 종료로 이동
	코드2;
}
~~~
break에 도달하면 코드2가 실행되지 않고 while문이 종료된다.
~~~java
While(조건식){
	코드1;
	continue; //즉시 조건식으로 이동한다.
	코드2;
}
~~~
contune를 만나면 코드2가 실행되지 않고 다시 조건식으로 이동한다.
조건식이 참이면 while문을 실행한다.

#### 입력 띄어쓰기로 받기

~~~java
String[] s = scanner.nextLine().split(" ");
~~~
띄어쓰기를 기준으로 입력을 나눠서 받음.


#### 향상된 for문
iter
~~~java
int[] numbers = {1,2,3,4,5}
for(int number : numbers){

}
~~~

###  String 배열에 문자 할당하기
~~~java
st[i] = string.charAt(i) + "";  
~~~

#### String 문자열의 길이 계산하기
~~~java
int a = string.length();
~~~

#### EOF (End of file)
~~~java
scanner.hasNext()
~~~
더 이상 입력이 없음.

#### String 숫자를 int 로 받아오기
~~~java
int num1 = Integer.parseInt(x);  //String x 의 숫자를 int 로 받아옴
~~~