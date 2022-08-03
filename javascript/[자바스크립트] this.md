## this의 정의

: `this`는 '자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수'다.



### 1. 단독으로 쓴 this

- Window를 가리킴



### 2. 함수 안에 쓴 this

- 함수의 주인에게 바인딩 됨
- 함수의 주인은 Window



### 3. 메서드 안에서 쓴 this

- 메서드 호출 시 메서드 내부 코드에서 사용된 this는 해당 메서드를 호출한 객채로 바인딩 됨

```javascript
let john = {
    firstName: "John",
    lastName: "Doe",
    driveCar() {
        console.log(this.firstName + " drives a car.")
    }
}

john.driveCar(); // John drives a car
```



### 4. 이벤트 핸들러 안에서 쓴 this

이벤트 핸들러에서 this는 이벤트를 받는 HTML 요소를 가리킴



### 5. 생성자 안에서 쓴 this

생성자 함수가 생성하는 객체로 this가 바인딩 됨

```javascript
function person() {
    this.firstName = "John",
    this.lastName = "Doe",
    this.start = function() {
        console.log(this.firstName + "drives a car.")
    }
}

let person1 = new person();

console.log(person1); // { firstName: 'John', lastName: 'Doe', start: [Function] }
```



### 6. 화살표 함수로 쓴 this

- 함수 안에서 this가 전역 객체가 되지 않게 할 때는 화살표 함수 이용

- 화살표 함수는 전역 컨텍스트에서 실행되더라도 this를 새로 정의 하지 않고 바로 바깥 함수나 클래스의 this를 사용

- 화살표 함수 사용 제약 :

  - 무조건 익명함수로만 사용할 수 있음

  - 메소드나 생성자 함수로 사용할 수 없음

- 화살표 함수 사용 이유 : **this 바인딩**에 차이



### 7. JavaScript 함수의 this 바인딩

1. 전역공간의 this : 전역객체

2. 메소드 호출 시 메소드 내부의 this : 해당 메소드를 호출한 객체

3. 함수 호출 시 함수 내부의 this : 지정되지 않음 (자바스크립트 설계상 오류)

   ```javascript
   const cat = {
     name: 'meow',
     foo1: function() {
       const foo2 = function() {
         console.log(this.name);
       }
       foo2();
     }
   };
   
   cat.foo1();	// undefined
   ```

   

   ```javascript
   const cat = {
     name: 'meow',
     foo1: function() {
       const foo2 = () => {
         console.log(this.name);
       }
       foo2();
     }
   };
   
   cat.foo1();	// meow
   ```

   *** 화살표 함수에는 this가 아예 없음!!

   화살표 함수로 선언한 함수에는 this가 없음 

   ```javascript
   JavaScript에서는 어떤 식별자(변수)를 찾을 때 현재 환경에서 그 변수가 없으면 바로 상위 환경을 검색
   ```

   

   

















