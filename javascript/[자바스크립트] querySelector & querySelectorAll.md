# [자바스크립트] querySelector & querySelectorAll

querySelector :  특정 name,id,class를 제한하지 않고 css선택자를 사용하여 요소를 찾습니다. 같은 id 또는 class 일 경우 스크립트의 **최상단 요소**만 로직에 포함합니다.

```+javascript
querySelector(#id) => id 값 id를 가진 요소를 찾습니다.
querySelector(.class) => class 값 class를 가진 요소를 찾습니다.
```



querySelectorAll : 배열과 비슷한 nodeList를 반환하기때문에

for 문으로 인덱스로 접근 가능