---
layout: post
title: 자바스크립트 동작원리- 엔진, 런타임, 호출 스택
featured-img: js1
---

# 자바스크립트의 동작원리: 엔진, 런타임, 호출 스택



V8엔진에 싱글 쓰레드 기반이며 콜백 큐를 사용한다.



V8엔진은 Chrome과 Node.js에서 사용한다.

![js-engine-structure](https://joshua1988.github.io/images/posts/web/translation/how-js-works/js-engine-structure.png)

엔진의 주요 두 구성요소

- Memory Heap: 메모리 할당이 일어나는 곳
- Call Stack: 코드 실행에 따라 호출 스택이 쌓이는 곳



## 런타임

자바스크립트의 대부분의 개발자들이 setTimeout과 같은 브라우저 내장 API를 사용하는데 이 API를 자바스크립트 엔진에서 제공하지는 않는다. 

![js-engine-runtime](https://joshua1988.github.io/images/posts/web/translation/how-js-works/js-engine-runtime.png)

자바스크립트 엔진 이외에도 자바스크립트에 관여하는 다른 요소들이 많다. DOM,Ajax,setTimeout과 같이 브라우저에서 제공하는 API들을 Web API라고 한다. 그리고 Event Loop와 Callback Queue가 있다.



## 호출스택

자바스크립트는 싱글 쓰레드 기반 언어이다. 호출 스택이 하나이기에 한 번에 하나의 작업만 처리할 수 있다.



호출 스택은 기본적으로 우리가 프로그램 상에서 어디에 있는지를 기록하는 자료구조이다. 함수를 실행하면, 해당 함수는 호출 스택의 가장 상단에 위치한다. 함수의 실행이 끝날 때는 해당 함수를 호출 스택에서 제거한다.

``` 스택 예
function multuply(x,y){
	return x*y;
}
function printSquare(x){
	var s=multply(x,x);
	console.log(s);
}
printSquare(5);
```

처음 엔진이 이 코드를 실행하는 시점에는 호출 스택이 비어있지만 코드가 실행되면서 호출 스택은 다음과 같이 변한다.

![call-stack](https://joshua1988.github.io/images/posts/web/translation/how-js-works/call-stack.png)

호출 스택의 각 단계를 스택 프레임이라고 한다.

그리고 보통 예외가 발생했을 때 콘솔 로그 상에서 나타나는 스택 트레이스(Stack Trace)가 오류가 발생하기까지의 스택 트레이스들로 구성된다. 에러가 났을 때의 호출 스택의 단계를 의미한다.

``` 스택 트레이스 예
function foo(){
	throw new Error('sessionStack will help you resolve craches :)');
}
function bar(){
	foo();
}
function start(){
	bar();
}
start();
```

위 코드가 foo.js에 있다고 하고 크롬에서 실행하면

![stack-trace-error](https://joshua1988.github.io/images/posts/web/translation/how-js-works/stack-trace-error.png)

시간 역순으로 읽으면 된다. 

먼저 첫줄, foo.js의 두번째 줄에서 에러가 발생했다. 

foo.js의 10번째 줄에서 start() 함수 호출

foo.js의 6번째 줄에서 bar()함수 호출

foo.js의 2번재 줄에서 foo()함수 호출

(스택 트레이스)



호출 스택이 최대 크기가 되면 "스택 날려 버리기"가 일어난다. 이는 반복문 코드를 광범위하게 테스트하지 않고 실행했을 때 자주 발생한다.

``` 스택 날려 버리기 예
function foo(){
	foo();
}
foo();
```

엔진에서 이 코드를 실행할 때, foo()에 의해서 foo함수가 호출된다. 그런데 여기서 foo함수가 재귀 호출을 수행한다. 그러면 매번 실행할 때마다 호출 스택에 foo()가 쌓이게 된다.

![maximum-call-stack](https://joshua1988.github.io/images/posts/web/translation/how-js-works/maximum-call-stack.png)

그러다가 특정 시점에 함수 호출 횟수가 호출 스택(Call Stack)의 최대 허용치를 넘게 되면 브라우저가 아래와 같은 에러를 발생시킨다.

![maximum-call-stack-error](https://joshua1988.github.io/images/posts/web/translation/how-js-works/maximum-call-stack-error.png)

싱글 쓰레드 기반 코딩은 멀티 쓰레드 환경에서 제기되는 복잡한 문제나 시나리오를 고민하지 않아도 되기에 상당히 쉽다. ex)데드락(운영체제 혹은 소프트웨어의 잘못된 자원 관리로 인해 둘 이상의 프로세스가 함께 멈춰 버리는 현상)

허나 싱글 쓰레드에서 코드를 실행하는 것은 상당한 제약이 있다.

 한 개의 호출 스택을 갖고 있는 자바스크립트의 실행이 느려지면 어떻게 될까?



## 동시성(Concurrency)&이벤트 루프(Event Loop)

호출 스택에 처리 시간이 어마어마하게 오래 걸리는 함수가 있으면 무슨 일이 발생할까? 예를 들어, 브라우저에서 자바스크립트로 매우 복잡한 이미지 프로세싱 작업을 한다 해보자.

여기서 문제는 호출 스택에서 해당 함수가 실행되는 동안 브라우저는 아무 작업도 못하고 대기 상태가 된다는 것이다. 브라우저는 페이지를 그리지도 못하고, 어느 코드도 실행을 못하는 가만히 있는 상태가 되버린다. 만약 매끄럽고 자연스러운 화면 UI를 가진 앱을 원한다면 위 경우는 문제가 된다.



또한 브라우저가 호출 스택의 정말 많은 작업들을 처리하다 보면 화면이 오랫동안 응답하지 않게 되는데, 이 경우에 대부분의 브라우저가 아래와 같은 에러를 띄우면서 페이지를 종료할 건지 물어본다.

![terminate-page-popup](https://joshua1988.github.io/images/posts/web/translation/how-js-works/terminate-page-popup.jpeg)

그렇다면 어떻게 페이지 렌더링 동작을 방해하지 않고 브라우저의 응답도 끊지 않으면서 연산량이 많은 코드를 실행할 수 있을까? 정답은 바로 비동기 콜백이다. 



## 참고

원문 : [How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)