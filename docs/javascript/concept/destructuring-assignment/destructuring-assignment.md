# 비구조화 할당(Destructuring Assignment)

## 배열 비구조화 할당

ES6의 배열 비구조화 할당은 배열의 각 요소를 배열에서 추출하여 1개 이상의 변수에 할당한다. 이때 **비구조화될 대상은 이터러블이어야 한다.**

```javascript
const arr = [1, 2, 3];

// ES6 배열 비구조화 할당
// 할당 기준은 배열의 index이다.
const [one, two, three] = arr;
console.log(one, two, three); // 1 2 3
```

배열 비구조화 할당을 위해서는 할당 연산자 왼쪽에 값을 할당받을 변수를 선언해야 한다. 이때 변수를 배열 리터럴 형태로 선언한다. 또한 이터러블을 할당하지 않으면 에러가 발생한다.

```javascript
const [x, y] = [1, 2];
const [x, y]; // SyntaxError: Missing initializer in destructuring declaration
const [a, b] = {}; // TypeError: {} is not iterable
```

배열 비구조화 할당의 기준은 배열의 index이다. 이때 변수의 개수와 이터러블의 요소의 개수가 반드시 일치할 필요는 없다.

```javascript
const [a, b] = [1, 2];
console.log(a, b); // 1 2

const [c, d] = [1];
console.log(c, d); // 1 undefined

const [e, f] = [1, 2, 3];
console.log(e, f); // 1 2

const [g, , h] = [1, 2, 3];
console.log(g, h); // 1 3
```

또한 변수에 기본값을 설정하는 것도 가능하다.

```javascript
// 기본값
const [a, b, c = 3] = [1, 2];
console.log(a, b, c); // 1 2 3

// 기본값보다 할당된 값이 우선된다.
const [e, f = 10, g = 3] = [1, 2];
console.log(e, f, g); // 1 2 3
```

배열 비구조화 할당을 위한 변수에 rest 파라미터와 유사한 rest 요소(rest element)를 사용할 수 있다. 이때 rest 요소는 반드시 마지막에 위치해야 한다.

```javascript
const [x, ...y] = [1, 2, 3];
console.log(x, y); // 1 [2,3]
```

## 객체 비구조화 할당

ES6의 객체 비구조화 할당은 객체의 각 프로퍼티를 추출하여 1개 이상의 변수에 할당한다. 이때 비구조화 할당의 대상은 객체여야 하며, **할당 기준은 프로퍼티 키**이다. 배열과는 다르게 순서는 의미가 없다.

```javascript
const user = { firstName: 'Sungbaek', lastName: 'Kim' };

const { lastName, firstName } = user;
console.log(firstName, lastName); // Sungbaek Kim
```

객체의 프로퍼티 키와 다른 변수 이름으로 프로퍼티 값을 할당받으려면 다음과 같이 변수를 선언해야 한다.

```javascript
const user = { firstName: 'Sungbaek', lastName: 'Kim' };

const { lastName: l_N, firstName: f_N } = user;
console.log(f_N, l_N); // Sungbaek Kim
```

객체 비구조화 할당을 위한 변수에 기본값을 설정할 수 있다.

```javascript
const { firstName = 'Sungbaek', lastName } = { lastName: 'Kim' };
console.log(firstName, lastName); // Sungbaek Kim

const { firstName: f_N = 'Sungbaek', lastName: l_N } = { lastName: 'Kim' };
console.log(f_N, l_N); // Sungbaek Kim
```

객체 비구조화 할당은 객체에서 프로퍼티 키로 필요한 프로퍼티 값만 추출하여 변수에 할당할 때 유용하게 사용할 수 있다.

```javascript
const str = 'Hello';
const { length } = str;
console.log(length); // 5

const todo = { id: 1, content: 'HTML', completed: true };
const { id } = todo;
console.log(id); // 1
```

객체 비구조화 할당은 객체를 인수로 전달받는 함수의 매개변수에도 사용할 수 있다.

```javascript
function printTodo({ content, completed }) {
  // ...
}
function printTodo(todo) {
  const { content, completed } = todo;
  console.log(`할일 ${content}은 ${completed ? '완료' : '미완료'} 상태입니다.`);
}
printTodo({ id: 1, content: 'HTML', completed: true }); // 할일 HTML은 완료 상태입니다.
```

배열의 요소가 객체인 경우 배열 비구조화 할당과 객체 비구조화 할당을 동시에 진행할 수 있다.

```javascript
const todos = [
  { id: 1, content: 'HTML', completed: true },
  { id: 2, content: 'CSS', completed: false },
  { id: 3, content: 'JS', completed: false }
];

// todos 배열의 두 번째 요소(객체)에서 id 프로퍼티만 추출
const [, { id }] = todos;
console.log(id); // 2
```

중첩 객체는 다음과 같이 사용한다.

```javascript
const user = {
  name: 'Kim',
  address: {
    zipCode: '12345',
    city: 'Seoul'
  }
};

// address 프로퍼티로 객체를 추출하고 해당 객체의 city 프로퍼티 값을 추출한다.
const {
  address: { city }
} = user;
console.log(city); // Seoul
```

2020년 7월 기준 TC39 프로세스의 stage 4(Finished) 단계에 제안되어 있는 Rest 프로퍼티를 사용할 수도 있다.

```javascript
const { x, ...rest } = { x: 1, y: 2, z: 3 };
console.log(x, rest); // 1 { y: 2, z: 3 }
```

## Reference

- [모던 자바스크립트 Deep Dive 36장 - 디스트럭처링 할당](http://www.yes24.com/Product/Goods/92742567)
