### 우아한 테크러닝 Day 5

## Redux
```javascript
import { createStore } from './redux';

function reducer(state = { count: 0 }, action) {
    switch(action, type) {
        case 'inc':
            return {
                ...state,
                counter: state.counter + 1,
            };
        default:
            return { ...state };
    }
}

const store = createStore(reducer);
//요걸 구독을 해야한다. reudx는 구독 모델이니까! => pub/sub

store.subscribe(() => {
    console.log(store.getState());
    //필요한 상태만 꺼내서 rendering..
}))

store.dispatch({
    type: 'inc'
});

store.dispatch({
    type: 'inc'
});

```

위 모든 동작은 *동기적*으로 작동한다.
이를 위해 reducer는 *'순수 함수'*여야하고, 같은 input에 대해 같은 output을 뱉는다.

반면 redux에서 순수하지 않은 함수를 실행해야하는 경우도 있는데, 대표적으로 비동기 함수 등이 있다.
eg) api 호출. -> 호출했을 때 성공할 수도 있고, 실패할 수도 있다.

그렇다면 이 순수하지 않은 상황(비동기)는 어떻게 다룰 것인가?

일단 위 구조 만으로는 불가능하다.


## Redux에서의 비동기 처리는?
```javascript
import { createStore } from './redux';

function api(url, cb) {
    setTimeout(() => {
        cb({ type: '응답이야!', date: [] });
    }, 2000);
}

function reducer(state = { count: 0 }, action) {
    switch(action, type) {
        case 'inc':
            return {
                ...state,
                counter: state.counter + 1,
            };
        case 'fetch-user':
            api('/api/v1/users/1', user => {
                return { ...state, ...users}
            });
            break;
        default:
            return { ...state };
    }
}

const store = createStore(reducer);

store.subscribe(() => {
    console.log(store.getState());
}))

store.dispatch({
    type: 'fetch-user'
})

```

비동기 코드의 이해를 위해 `api` 함수를 작성하였다.

위 코드에서, 상태에 대한 변화는 callback 안에 들어가있다.
위 로직은 동기적으로 동작하기 때문에 api 함수가 끝날 때까지 기다리지 않는다. 그래서 reducer는 api 함수를 호출한 후 break 문으로 넘어가 끝나고, 결국 상태에 반영되지 않는다.  
그 후 callback이 실행되어서 return 된다고해도 작업을 요청했을 때의 state 상태와 callback이 실행되었을 때의 state는 다를 것이다.   
그래서 이 상태에서 비동기 작업은 불가능하다!(reducer가 순수함수이니까)

그래서 redux는 이를 *'미들웨어'*를 통해 처리한다.  
동기적 작업은 reducer 안에서 처리하고,  
사이드 이펙트가 우려되는 작업들은 reducer 밖, 그 중 '미들웨어'에서 처리하게 한다.

```javascript
const myMiddleware = store => dispatch => action => {
    dispatch(action);
}

function yourMiddleware(store) {
    return function(dispatch) {
        return function(action) {
            dispatch(action);
        }
    }
}

function ourMiddleware(store, dispatch, action) {
    dispatch(action);
}

console.log('log => ', 'inc');
myMiddleware(store)(store.dispatch)({ type: 'inc' });

console.log('log => ', 'inc');
ourMiddleware(store, store.dispatch, { type: 'inc' });

```

>middleware란: 데이터의 흐름의 중간에 꽂혀서 데이터를 처리해주고 반환해주는 것.

my, yours와 our의 다른 점:  
my, yours는 인자 하나하나로 중첩되어 있고(클로저) our는 한번에 다 받는다.  
my, yours 같은 함수를 커링이라고 부른다.(인자 여러개마다 함수를 쪼개는 것.)  
*=> 왜 미들웨어에서 커링을 사용하는 걸까?*

만약 dispatch 하기 전에 어떤 type을 보내는지 console.log를 찍고 싶다고 하자.  
실제 production mode에서 redux는 우리가 수정 할 수 없으니, dispatch 코드마다 쫒아가며 console.log를 찍어야 할 것이다.  
어떻게 하면 멋지게 log를 찍는 코드를 작성 할 수 있을까?

## 몽키패치

Redux 공식 페이지에서 [미들웨어에 대한 설명](https://ko.redux.js.org/advanced/middleware)을 인용하여 적었다.

```javascript
let next = store.dispatch
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

이렇게하면 이제 `dispatch` 함수를 실행 할 때마다 log가 남겨질 것이다.  
그렇지만.. 만약 `dispatch`를 실행할 때마다 위와 같은 작업을 n개 적용하고 싶다며 어떻게 해야할까?  
여기서는 log를 남기는 작업과 error 발생시 report하는 작업 2개가 필요하다고 가정한다.


```javascript
function patchStoreToAddLogging(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}

function patchStoreToAddCrashReporting(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndReportErrors(action) {
    try {
      return next(action)
    } catch (err) {
      console.error('Caught an exception!', err)
      Raven.captureException(err, {
        extra: {
          action,
          state: store.getState()
        }
      })
      throw err
    }
  }
}
```

위와 같이 각각의 작업을 함수로 분리하여 작성하였다.  
코드를 보면 구조적으로 next(action) 함수를 return 하고 있는 것을 볼 수 있다.  

``` javascript
patchStoreToAddLogging(store)
patchStoreToAddCrashReporting(store)
```

그럼 이 두 함수를 어떻게 `dispatch` 함수 안으로 숨길 수 있을까?

```javascript
function logger(store) {
  let next = store.dispatch

  // 앞에서:
  // store.dispatch = function dispatchAndLog(action) {

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

```javascript
function applyMiddlewareByMonkeypatching(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  middlewares.forEach(middleware => (store.dispatch = middleware(store)))
}
```

이렇게하면 각각의 미들웨어를 store의 dispatch 함수에 적용할 수 있다.

```javascript
applyMiddlewareByMonkeypatching(store, [logger, crashReporter])
```

사용은 위와 같이 하면 된다.

여기서 한번 더 나아가보자.  
우리는 dispatch를 덮어씌우지 않고 바로 반환하게 할 수도 있다.
```javascript
function logger(store) {
  return function wrapDispatchToAddLogging(next) {
    return function dispatchAndLog(action) {
      console.log('dispatching', action)
      let result = next(action)
      console.log('next state', store.getState())
      return result
    }
  }
}
```

이전 코드에서는 store를 받은 후  `let next = store.dispatch`로 원본 dispatch를 저장 후 덮어씌우도록 했지만, 위 코드에서는 바로 `next`라는 파라미터로 받고 있다.

```javascript
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

그리고 ES6의 화살표 문법을 사용하면 *커링*이 더 보기 쉬워진다.
Redux의 미들웨워는 위와 같이 커링을 활용하여 작성되었다고 한다.


## 커링
```javascript
const add1 = function(a, b) {
    return a + b;
}

const add2 = function(a) {
    return function(b) {
        return a + b;
    }
}

add1(10, 20);
add2(10)(20);
```
`add1`의 경우는 사용자가 개입할 여지가 없다.(바로 함수 호출 해버리니까)

그렇지만 `add2`의 경우는 (10)과 (20) 사이에 사용자가 개입할 수 있다. 왜냐면 parameter 마다 함수로 분리되어 있기 때문이다.

그래서 아래와 같은 몽키패치가 가능하다!

```javascript
const addTen = add2(10);//함수 합성, 생성, 조합 등 다양한 이름으로 불린다

//do some...

addTen(20);
addTen(120);
```

`add2` 같은 함수를 커링이라고 하는데, 커링은 인자와 인자 사이에 개입할 수 있는 여지를 열어주는 프로그래밍 테크닉이다.


## Redux에서의 미들웨어 처리
```javascript
const logger = (store) => (next) => (action) => {

}
```

제품 상세를 보는 액션이 왔는데, 얘는 로그인이 되어있어야 접근 가능한데! 하고 store에서 유저 정보 확인해보고 로그인 안되어있으면 로그인 페이지로 이동하는 등의 비즈니스 로직 활용이 가능하다.  

## Q&A
Q. useCallback과 useMemo를 사용하는 기준이 궁금합니다.  
A. hook 보면서 보죵
https://kentcdodds.com/blog/usememo-and-usecallback

Q. 리액트에서 hook을 사용할 때 클로저가 발생한다고 하셨는데, 조금 더 설명해주실 수 있을까요?  
A. https://rinae.dev/posts/getting-closure-on-react-hooks-summary

Q. async function에서도 generator를 사용하던데 그 경우는 어떤 경우인가요?  
A. generator가 훨씬 더 일반화된 함수이고, async 함수는 그 안에서 yield 되는 애들이 promise인 형태.
근데 저 둘이 같이 쓰이는 경우는.. 거의 안 좋은 경우 밖에 못 봤다.

Q. 여러개 API를 여러번 요청해야하는데 어떻게 해야할까요?  
A. UI/UX 상황에 따라 다를 것 같다.
분할 요청 할 수 있다면 묶어서 요청하는게 좋을 것 같다.
근데 묶어서 보내는 것도 Http 요청인데다가, 브라우저마다 동시 요청에 대한 제약이 있다.
서버에 gateway 같은게 있다면 gateway와 webapp 간에는 요청을 묶어서 한번에 받고 서버에서 라운드 로빈 방식으로 처리하는 등.. UX 상황과 여러가지를 고려해서 처리해야 할 것 같다.

Q. api 호출하는 부분을 리듀서 바깥에서 호출하면 되는 거라면, 굳이 미들웨어를 쓸 필요가 있는지 모르겠습니다  
A. 다른 작업으로 api를 호출해도 되지만, 그렇게되면 일관성이 사라진다.
일관성이 흐트러지면, 그 맥락을 모르는 개발자가 보았을 때는 이해하기 힘들어질 수 있다.