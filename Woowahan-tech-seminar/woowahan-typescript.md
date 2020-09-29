# 우아한 타입스크립트

## 목표
Typescript로 타이핑을 잘하면, 런타임 전에 미리 알 수 있는 오류도 있다.

## interface와 alias
structual type system - 구조가 같으면, 같은 타입이다.
nominal type system - 구조가 같아도 이름이 다르면, 다른 타입이다.

- union types
type에만 지원되는 기능이다.
```typescript
type PetType = Cat | Dog;

interface IPet extends PetType {

}//error

class Pet extends PetType {
    
}//error

```

- Declaration Merging
interface에만 지원되는 기능이다.

```typescript
interface IMerging {
    a: string
}

interface IMerging {
    b: string
}

let mi: IMerging; // a, b
```

## 서브 타입과 슈퍼 타입
: subtype은 집합의 관계에서 포함되는 것을 subtype 이라 부른다.  
supertype은 집합의 관계에서 포함하는 것을 supertype 이라 부른다.

```typescript
let sub1: 1 = 1;//literal type은 1이기 때문에 2,3 등을 넣을 수는 없다.
let sup1: number = sub1; // sub1은 sup1의 서브타입이다. (1은 number에 포함되므로.)

let sub2: number[] = [1];
let sup2: object = sub2;
sub2 = sup2;//error

let sub3: [number, number] = [1, 2];
let sup3: number[] = sub3;
sub3 = sup3;//error

let sub5: never = 0 as never;
let sup5: number = sub5;
sub5 = sup5;//error

class SubAnimal{}
class SubDog extends SubAnimal{
    eat() {}
}

let sub6: SubDog = new SubDog();
let sup6: SubAnimal = sub6;
sub6 = sup6;//error
```

1. 같거나 서브 타입인 경우, 할당이 가능하다. => 공변
- primitive type
- object
갹갹의 프로퍼티가 대응하는 프로퍼티가 같거나 서브타입이어야 한다.
- array
object와 마찬가지.

2. 함수의 매개변수 타입만 같거나 슈퍼타입인 경우, 할당이 가능하다. => 반병
```typescript
class Person{}
class Developer extends Person{
    coding() {}
}
class StartupDeveloper extends Developer{
    burning() {}
}

function tellme(f: (d: Developer) => Developer) {}

//할당 가능
tellme(function dToD(d: Developer): Developer {
    return new Developer();
});

//d의 Person이 Developer의 슈퍼타입이므로 할당 가능하다.
tellme(function pToD(d: Person): Developer {
    return new Developer();
});

//d의 StartupDeveloper는 Developer의 슈퍼타입이 아니므로 에러가 발생할 수 있다.
tellme(function sToD(d: StartupDeveloper): Developer {
    return new Developer();
})
```

strictFunctionTypes 옵션을 켜면, sToD와 같은 경우에 대해 경고를 날려준다.

> any 대신 unknown을 쓰는게 훨씬 더 안정성을 높힌다. any는 모든 것에 가능성을 열어두는 것이고, unknown은 반대로 모든 가능성을 닫고 시작한다.

## Type Guard로 안전함을 파악하기
1. typeof Type Guard - 보통 primitive 타입일 경우
```ts
function getNumber(value: number | string): number {
    value; // number | string
    if (typeof value === 'number') {
        return value;//number
    }
    value;//string
    return -1;
}
```

2. instanceof Type Guard - Error 객체 구분에 많이 쓰인다
```ts
class NegativeNumberError extends Error{}

function getNumber(value: number): number | NegativeNumberError {
    if (value < 0) return new NegativeNumberError();
    return value;
}

function main() {
    const num = getNumber(-10);
    
    if (num instanceof NegativeNumberError) {
        return;
    }

    num; // number
}
```
-> 백엔드 코드 짤 때 이런식으로 몇번 처리했던 기억이 난다.

3. in operator Type Gurad - object의 프로퍼티 유무로 처리하는 경우
```ts
function redirect(user: Admin | User) {
    if ("role" in user) {

    } else {

    }
}
```

4. literal Type Guard - object의 프로퍼티가 같고, 타입이 다른 경우 사용하기 용이하다.

```ts
function getWhellOrMotor(machine: Card | Boat): number {
    if (machine.type === 'CAR') {//마치 리듀서 처럼!

    } else {

    }
}
```

5. custom Type Guard
```ts
function isCar(arg: any): arg is Car {
    return arg.type === 'CAR';
}

isCar(someInstance);
```

