# undeclared & undefined & null

## undeclared(미선언 변수)

```javascript
a = 1;

console.log(typeof a) //undefined
```

생성되지 않은 변수에 값을 할당 할 때 생긴다. Undeclared 변수는 현재 스코프 외부에 전역으로 설정된다.
(`strict` 모드에서는 undeclared 변수에 값을 할당하려고하면 `ReferenceError`가 발생한다.)

type 검사를 하면 undefined로 나온다.

## undefined
```javascript
var a;

console.log(typeof a) //undefined
```

스코프에 변수가 선언되었으나 아무런 값도 설정되지 않은 상태이다.

## null
```javascript
var a = null;

console.log(typeof a) //object
```
null의 경우는 변수에 null 값을 명시적으로 할당한 경우 생긴다.
이는 어떤 값이 의도적으로 비어있음을 표현한 것으로, 어떠한 객체도 가리키고 있지 않음을 표시한다.

## null vs undefined
```javascript
const a;
const b = null;

console.log(typeof a == typeof b) //true
console.log(typeof a === typeof b) //false
```

undefined는 값이 할당되지 않은 것이지만, null은 명시적으로 값을 할당한 것이다.
또한 둘은 추상 평등 연산자로 검사하면 true를 반환하므로 완전 항등 연산자로 검사해야한다.