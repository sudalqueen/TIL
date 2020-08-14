# 타입 추론, Const Assertion

## 타입 추론
정적 언어에는 보통 '타입 추론' 기능이 포함되어 있다.
'타입 추론'은 변수에 대입하는 '리터럴의 타입'을 보고 해당 변수의 타입을 자동으로 지정해준다.

Typescript에서는 let과 const에서의 타입 추론이 다르게 동작한다.

```typescript
let hello = 'world';
```

위 코드에서 hello 변수의 타입은 타입 추론에 의해 'string' 으로 지정된다.

```typescript
const first_name: string = 'sudal';
const last_name = 'kim'; // 리터럴 타입
```

위 코드에서 first_name은 명시한 타입인 'string'으로 할당되지만, 아래의 last_name은 타입 추론에 의해 "kim"이 타입이 된다.

왜냐면 const는 값의 재할당이 불가능하기 때문에 타입 없이 선언될 경우 타입 추론에 의해 할당된 값이 타입이 된다.

IDE에서 last_name 변수를 확인해보면 `const last_name: 'kim'` 이렇게 뜰 것이다.

## Const Assertion

Typescript 3.4 버전에서 추가된 기능으로, 위 코드에서 `as const`를 *const assertion* 이라고 한다.
React + Redux + Typescript로 프로젝트를 하다보면 액션 타입 정의문에서 많이 보이는 친구다.

const assertion을 사용하면 const 변수를 사용할 때와 같은 타입 추론 규칙을 적용할 수 있다. 즉, 타입을 강제 할 수 있다!

```typescript
const circle = {
    type: 'circle',
    radius: 10
};
```

위 코드에서 circle의 type을 확인해보면 아래와 같이 나올 것이다.

```typescript
declare const circle: {
    type: string;
    radius: number;
};
```
이 경우 circle.type은 string이기 때문에 타입에 'circle'을 정의한게 의미가 없어진다.

하지만 `circle` 변수 정의문의 마지막에 `as const`를 붙여준다면 아래와 같은 타입으로 확인 할 수 있다.

```typescript
declare const circle: {
    readonly type: "circle";
    readonly radius: 10;
};
```

---
참고
https://feel5ny.github.io/2017/11/18/Typescript_05/
https://medium.com/@seungha_kim_IT/typescript-3-4-const-assertion-b50a749dd53b