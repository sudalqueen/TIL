# Event

### Event Bubbling
이벤트 버블링이란 특정 화면 요소에서 이벤트가 발생했을 때, 해당 이벤트가 더 상위의 화면 요소들로 전달되어 가는 특성이다.

### Event Capture
이벤트 버블링과는 반대되는 전파 방식으로, 이벤트가 발생되었을 때 최상위 요소에서 해당 태그를 찾아 내려가는 방식이다.

### Event Delegation
이벤트 위임은 하위 요소 각각에 이벤트를 붙이지 않고, 상위 요소에서 하위 요소의 이벤트를 제어하는 방식이다.
JS에서 자주 쓰이는 이벤트 핸들링 패턴이다.

동적으로 DOM이 추가/삭제 되는 코드에서 DOM이 추가될 때마다 일일이 이벤트를 새로 걸어주는 것은 번거롭고 비효율적인 작업이다.
이를 이벤트 위임을 통해 해결할 수 있다.

---
참고

https://ko.javascript.info/event-delegation

https://joshua1988.github.io/web-development/javascript/event-propagation-delegation/