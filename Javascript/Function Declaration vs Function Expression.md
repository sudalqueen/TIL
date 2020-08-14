# Function Declaration vs Function Expression

React + Typescript로 개발할 때 항상 함수 표현식을 사용해왔는데 최근 함수 선언식을 쓰다가 정리해보는 문서.
단순히 문법적 차이만 있는 것이 아닌데, 자꾸 잊어먹고 개발하게 되어서..

### Function Declaration(함수 선언식)

    function HelloWorld(): React.ReactNode {
    	return <h1>Hello World!</h1>
    }
위는 일반적인 프로그래밍 언어에서의 함수 선언과 비슷하다.

### Function Expression(함수 표현식)

    const HelloWorld: React.FC<props> = () => {
    	return <h1>Hello World!</h1>
    }

함수 선언식은 호이스팅에 영향을 받지만, 함수 표현식은 호이스팅에 영향을 받지 않는다.
또한 함수 표현식은 콜백으로도 사용 가능하다.

둘 중 어느게 더 좋은가? 라고 한다면 열이면 열 함수 표현식!을 꼽는다.
호이스팅 영향을 받지 않는게 큰 것 같다.

 [airbnb의 JS 스타일 가이드](https://github.com/airbnb/javascript#functions--declarations)를 참고해보면, 그냥 함수 표현식도 Bad case고 **함수 이름이 있는 함수 표현식**을 권장하고 있다. (에러 콜 스택이 더 깔끔해져서.)

---
참고 자료
https://joshua1988.github.io/web-development/javascript/function-expressions-vs-declarations/