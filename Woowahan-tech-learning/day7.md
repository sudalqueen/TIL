# 우아한 테크러닝 Day 7

### Promise

Promise에서 비동기 처리를 확인해보자

```javascript
const p1 = new Promise((resolve) => {
    setTimeout(() => {
        console.log('@4초 완료')
        resolve('4초 작업 완료')
    }, 4000)
});

const p2 = new Promise((resolve) => {
    setTimeout(() => {
        console.log('@2초 완료')
        resolve('2초 작업 완료')
    }, 4000)
});

p1.then((result) => {
    console.log(result);//4초 작업 완료
}).then((result) => {
    console.log(result);//undefined
})

```

then이란 resolve된 상태를 읽기 위한 도구, 인터페이스 인 것이다.
Promise는 상태 머신(state machine)이다.

위 코드에서 Promise는 이미 실행되었다.

```javascript
const p1 = () = new Promise((resolve) => {
    setTimeout(() => {
        console.log('@4초 완료')
        resolve('4초 작업 완료')
    }, 4000)
});

const p2 = () = new Promise((resolve) => {
    setTimeout(() => {
        console.log('@2초 완료')
        resolve('2초 작업 완료')
    }, 4000)
});

p1().then((result) => {
    console.log(result)
})
```

위 코드에서는 p1 함수를 실행시킨 후에 then을 연결하고 있다.
이렇게 작성하면, promise가 실행되는 시점과 then이 연결되는 시점이 일치해보인다.  
첫 번째 코드 처럼 작성한다면 시각적으로 보았을 때, 시점이 일치되지 않아서 헷갈릴 수도 있다.  

> pollyfill 이란?  
내부는 다른 코드로 구성되어 있지만 마치 동일한 코드로 구성되어 있는 것 처럼 도와주는 도구.

### 컴포넌트는 어떻게 나누는게 좋을까?
외부 상태에 의존적인 컴포넌트와 의존적이지 않은 컴포넌트로 분리한다.  

외부 상태에 의존적이지 않는 컴포넌트가 어딨어?
-> 여기서 '의존적'은 비즈니스 로직이고, 이 비즈니스 로직은 상태를 변경하는 것을 이야기한다.  

비즈니스 로직을 가지고 있는 컴포넌트를 다양한 상황에 따라 분리한다.  

*컨테이너 컴포넌트*
상태를 가져오고, 상태 데이터 처리하는 로직이 모아져 있다.
UI를 가지지 않으므로 StyledComponent등도 적용하지 않는다.
eg) 하나의 페이지를 구성하는 부분도 컨테이너 컴포넌트이다.
이 페이지 안에는 다시 여러개의 컨테이너 컴포넌트로 이루어질 것이다.

*플로우 컴포넌트*
사용자 흐름을 가지는 컨테이너 컴포넌트.
페이지 안에 여러개의 플로우가 있을 수 있다.

*프레젠테이셔널 컴포넌트*
받아온 상태를 뿌려서 보여주기만 한다.
-> 외부와 관련되어서 상태가 엮이면 더는 순수한 컴포넌트가 아니고, 분리해줘야한다.


1. 코드 스플리팅
- 사용자가 어떤 동선을 타고 많이 오는지, 어떤 컴포넌트가 dependency를 가지고 있는지 등을 보고 분리해야 할 것.
2. SSR을 선택하는 방법
- 코드 스플리팅을 상대적으로 덜 걱정하면서, 초기 로딩에 대한 부담을 덜 가질 수 있다.
다만 이쪽을 선택하면 서버쪽의 부하를 신경써야 할 것이다.

### 폴더 및 파일 구조
각 폴더 마다 index 파일을 하나 두고, index에서 모든 파일을 내보낸다. => 캡슐화!  
바깥쪽에서는 이름으로만 이들을 보고, 내부 폴더 구조는 숨길 수 있다.
내부 폴더 구조를 숨길 수 있으므로 외부의 상태와는 상관 없이 내부 폴더 구조를 리팩토링하기에도 쉽다.  

파일명의 경우 해당 파일이 가지는 역할과 책임을 나타낼 수 있는 이름으로 짓는다.  

### Typescript in Component
1. state 구조를 정의하는 부분을 interface, type alias 중 어느걸로 정의 할 것인가?  

```javascript
type Person = {
    name: string;
    age: number;
    job?: [];
}

interface Human {
    name: string;
    age: number;
    job?: [];
}

type box = number | string;//요게 union type
let b: box = 10;
```

interface, type alias가 기능적으로 거의 유사하다.  
interface는 상속을 지원해주고, union type을 지원하지 않는다. 반면 type은 union type을 지원해준다.  

아무래도 기능상 비슷하기 때문에, 필요에 따라 원하는걸 쓰면 될 것 같다.

```javascript
const p: Person = JSON.parse(prompt('객체를 입력해주세요'));
```

typescript를 쓴다고 해서 타입 에러를 모두 막을 수 있는 것은 아니다. 위의 코드와 같은 경우, run time 상황에서 사용자가 Person 객체에 맞는 정보를 입력할지 아닐지 모르기 때문이다.  
그렇기 때문에 run time, compile time에서 어떤 일이 일어나는지 명확히 알고 있어야한다.  

> 리액트와 OOP  
리액트의 class는 OOP를 구성하기 위해 만들어진 것이 아니다. 컴포넌트의 라이프사이클을 구성하기 위해 만들어진 것이다. 그래서 우리는 리액트에서 class를 만들고 생성자(new)로 쓴 적이 없다. 리액트 class 내부의 모든 함수는 public 하다. 그래서 리액트의 class로 OOP를 구성하기는 힘들다.(숨길거 숨기고 보여줄거 보여줘야하는데 다 public이라..) 그런데, 리액트 + mobx 조합의 경우는 OOP적으로 활용하기 좋은 조합이다. 여기에 타입스크립트 기능까지 함께하기 좋다. 


jsx에서..
```jsx
{
    this.props.isVisible ? <SomeComponent> : null
}
```

이런 코드는 가독성을 떨어뜨린다.
아래와 같이 코딩하는게 더 좋다.

```jsx
<Maybe>
    <Somecomponent>
</Maybe>
```

`<Maybe/>` 컴포넌트 내부에 삼항연산자가 들어가있다.  

### QnA
Q. 한 컴포넌트가 store로 액션을 dispatch하고, store에서 데이터를 받아와서 보여주는 것만 하는 컴포넌트는 컨테이너 컴포넌트인가요? 아니면 순수한 컴포넌트인가요?  
A. 강사님 기준, 컨테이너 컴포넌트! 라고 하셨다. UI를 표시하기 위한 data는 redux든 mobx든 쓸 수 있음. 이런 경우, redux를 쓰다가 mobx를 쓰는걸로 바뀌게 되는 식이라면 외부 dependency를 가지는 컴포넌트가 된다.
컴포넌트가 외부 dependency를 가지게 된다면, 외부 라이브러리가 바뀌는 경우 컴포넌트도 바뀌어야 할 것. 따라서 위의 경우는 container 컴포넌트이다!

Q. 하나의 컨테이너 안에 여러개의 컴포넌트가 있고, 이 때 컨테이너 컴포넌트의 state가 변하면 하위에 속한 모든 컴포넌트들이 모두 rerendering 될텐데 이 경우 신경 쓰시나요?  
A. 변경된 props에 해당하는 컴포넌트만 리랜더링 되도록 최적화해야한다.  

Q. 컨테이너 안에 컴포넌트 안에 컨테이너가 있다면..?  
A. 잘못된 설계 같습니다.  


<!-- 상태관리 immutable하게 하기 위해 redux를 썼고~ redux의 이러이러한거를 이용해서 요런 코드를 만들어서 해당 과제를 달성하였다~  -->

---
참고  
https://codesandbox.io/s/ordermonitor08-1ttxf?file=/src/sagas/index.ts
