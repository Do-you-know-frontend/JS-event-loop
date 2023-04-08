# JavaScript Event Loop

# JavaScript Event Loop

이벤트 루프는 브라우저에 내장되어 있는 기능 중 하나다.

자바스크립트는 싱글 스레드로 동작하기 때문에 한 번에 하나의 태스크만 처리할 수 있다. 하지만 브라우저가 동작하는 것을 살펴보면 많은 태스크가 동시에 처리되는 것처럼 느껴진다

**이처럼 자바스크립트의 동시성을 지원하는 것이 바로 이벤트 루프(event loop)다.**

1. 비동기로 진행되는 함수 **① setTimeout/ setInterval ② HTTP 요청 ③ 이벤트 핸들러** 은 브라우저에 의해 태스크 큐로 이동하게 되는데, 동기적으로 실행되는 함수가 모두 실행되고 콜 스택에서 팝된 이후에 태스크 큐에 먼저 들어온 함수부터 차례대로 콜스택에 푸시한다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/de0b9655-a3d2-40f5-aa9c-df65b491ebad/Untitled.png)

**① 콜 스택 (call stack)**

- 소스코드(전역 코드 및 함수 코드 등) 평가 과정에서 생성된 실행 컨텍스트가 추가되고 제거되는 스택 자료구조인 실행 컨텍스트 스택이 바로 콜 스택이다.
- 함수를 호출하면 함수 실행 컨텍스트가 순차적으로 콜 스택에 푸시되어 순차적으로 실행된다. 자바스크립트 엔진은 단 하나의 콜 스택을 사용하기 때문에 최상위 실행 컨텍스트(실행 중인 실행 컨텍스트)가 종료되어 콜 스택에서 제거되기 전까지는 다른 어떤 태스크도 실행되지 않는다.

**②힙 (heap)**

- 힙은 객체가 저장되는 메모리 공간이다. 콜 스택의 요소인 실행 컨텍스트는 힙에 저장된 객체를 참조한다.
- 메모리에 값을 저장하려면 먼저 값을 저장할 메모리 공간의 크기를 결정해야 한다. 객체는 원시 값과는 달리 크기가 정해져 있지 않으므로 할당해야 할 메모리 공간의 크기를 런타임에 결정(동적 할당)해야 한다. 따라서 객체가 저장되는 메모리 공간인 힙은 구조화되어 있지 않다는 특징이 있다.

이처럼 콜 스택과 힙으로 구성되어 있는 자바스크립트 엔진은 단순히 태스크가 요청되면 콜 스택을 통해 요청된 작업을 순차적으로 실행할 뿐이다. 비동기 처리에서 ① 소스코드의 평가와 ② 실행을 제외한 모든 처리는 자바스크립트 엔진을 구동하는 환경인 브라우저 또는 Node.js가 담당한다.

예를 들어,

① 비동기 방식으로 동작하는 setTimeout의 콜백 함수의 평가와 실행은 자바스크립트 엔진이 담당하지만

② 호출 스케줄링을 위한 타이머 설정과 콜백 함수의 등록은 브라우저 또는 Node.js가 담당한다

이를 위해 브라우저 환경은 태스크 큐와 이벤트 루프를 제공한다

**③ 태스크 큐 (task queue/event queue/callback queue)**

- setTimeout이나 setInterval과 같은 비동기 함수의 콜백 함수 또는 이벤트 핸들러가 일시적으로 보관되는 영역이다
- 태스크 큐와는 별도로 프로미스 후속 처리 메서드의 콜백 함수가 일시적으로 보관되는 마이크로태스크 큐도 존재한다

**④ 이벤트 루프 (event loop)**

- Event loop가 call stack, render 시퀀스,micro task queue, task queue를 굉장히 빠른 속도로 반복해서 확인한다.
- call stack에 함수가 들어오면 그 함수가 끝날때가지 call stack에 머물며 들어온 함수를 실행한다.
- 여기서 브라우저가 정한 렌더 업데이트 시간인 60프레임(1/60초)가 되지 않으면 render 시퀀스를 건너뛴고 돌게 된다.
- 만약 콜 스택이 비어 있고 micro task queue에 대기 중인 함수가 있다면 이벤트 루프는 순차적(FIFO)으로 micro task queue에 대기 중인 함수를 콜 스택으로 이동시킨다( 처리하던 도중에 micro task queue에 일이 들어오면 그 일도 같이 처리한다.)
- 이후 event loop는 task queue로 가고 여기에 있는 일들 중 1개만 call stack으로 보낸다.
- 다시 call stack으로가 일을 처리하고 render 시퀀스로 가게 된다.
- 이때 render 시퀀스의 일들을 update할 때가 되었다면 Request Animation Frame의 callback들을 처리하고 렌더링 순서에 따라 render tree를 만들고 layout을 짜고 paint를 해 다시 화면을 그린다.

**⑤ Web api**

- 브라우저에서 자체 지원하는 api입니다. Web api는 Dom 이벤트, Ajax (XmlHttpRequest), setTimeout 등의 비동기 작업들을 수행할 수 있도록 api를 지원합니다.

```jsx
function foo() {
  console.log('foo');
}

function bar() {
  console.log('bar');
}

setTimeout(foo, 0);
bar();

//실행 순서는?
```

- 정답
    
    ```jsx
    >>>
    bar
    
    foo // 0초(실제는 4ms) 후에 foo 함수가 호출된다.
    ```
    
    1. 전역 코드 실행 단계에서 위에 상대적으로 위에 존재하는 setTimeout 함수를 호출합니다
    - 이때 setTimeout 함수의 실행 컨텍스트가 생성되고 콜스택에 푸시되어 실행 중인 실행 컨텍스트가 됩니다
    1. setTimeout 함수가 실행되면서 콜백 함수를 호출 스케줄링 (주어진 시간 0ms)하고 종료되어 콜 스택에서 팝됩니다
    - 이때 타이머의 설정과 타이머가 만료되면 콜백 함수(foo)를 태스크 큐에 푸시하는 것은 브라우저의 역할입니다
    1. 브라우저가 수행하는 **4.1** 과 자바스크립트 엔진이 수행하는 **4.2** 는 병렬 처리됩니다
        
        4.1 브라우저는 타이머를 설정하고 타이머의 만료를 기다린다. 이후 타이머가 만료되면 콜백 함수 foo가 태스크 큐에 푸시됩니다
        
        4.2 bar 함수가 호출되어 bar 함수의 함수 실행 컨텍스트가 생성되고 콜 스택에 푸시되어 현재 실행 중인 컨텍스트가 된다. 이후 bar 함수가 종료되어 콜 스택에서 팝된다.
        
    2. 비동기로 진행되는 함수 **① setTimeout/ setInterval ② HTTP 요청 ③ 이벤트 핸들러** 은 브라우저에 의해 태스크 큐로 이동하게 되는데, 동기적으로 실행되는 함수가 모두 실행되고 콜 스택에서 팝된 이후에 태스크 큐에 먼저 들어온 함수부터 차례대로 콜스택에 푸시한다

```jsx
setTimeout(() => consolelog(1), 0);

Promise.resolve()
  .then(() => consolelog(2));
  .then(() => consolelog(3));

//실행 순서는?
```

- 정답
    
    2 → 3 → 1
    
    프로미스의 후속 처리 메서드의 콜백함수는 태스크 큐가 아니라 마이크로태크스 큐에 저장.
    
    마이크로태스크 큐는 태스크큐보다 우선순위가 높다.
    

**자바스크립트는 싱글 스레드 방식으로 동작한다. 이때 싱글 스레드 방식으로 동작하는 것은 브라우저가 아니라 브라우저에 내장된 자바스크립트 엔진이라는 것에 주의하기 바란다. 만약 모든 자바스크립트 코드가 자바스크립트 엔진에서 싱글 스레드 방식으로 동작한다면 자바스크립트는 비동기로 동작할 수 없다. 즉, 자바스크립트 엔진은 싱글 스레드로 동작하지만 브라우저는 멀티 스레드로 동작한다.**

javascript 런타임은 비동기 처리를 지원하지 않는다. 비동기 처리는 Javascript 엔진을 구동하는 런타임(Runtime) 환경에서 담당한다. 여기서의 런타임 환경이란, 브라우저나 Node.js를 말한다.

* 싱글스레드: 하나의 프로세스에서 하나의 스레드 실행

****Node 구성 요소****

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d32f381f-705b-41c4-95cc-2d4f4ff27ab3/Untitled.png)

****Node는 싱글 스레드다. 그러나... (libuv의 thread pool)****

****전체적인 Node의 동작 과정****

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c2b0b470-4ae7-4bd9-ba28-62ad4604fcef/Untitled.png)

[[TIL] 모던 자바스크립트 Deep Dive - 비동기 프로그래밍](https://velog.io/@hang_kem_0531/TIL-%EB%AA%A8%EB%8D%98-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-Deep-Dive-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)

[⭐️🎀 JavaScript Visualized: Promises & Async/Await](https://dev.to/lydiahallie/javascript-visualized-promises-async-await-5gke)

[Row level Node : js의 동작 방식부터 libuv와 event loop까지](https://darrengwon.tistory.com/953)

# 프로세스와 스레드
