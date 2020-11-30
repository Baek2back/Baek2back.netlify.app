# spread 문법

ES6에서 도입된 spread 문법은 하나로 뭉쳐 있는 여러 값들의 집합을 펼쳐서 개별적인 값들의 목록으로 만든다. spread 문법의 대상은 `for...of` 문으로 순회할 수 있는 이터러블에 한정된다.

```javascript
// ...[1,2,3]은 [1,2,3]을 개별 요소로 분리한다.
console.log(...[1, 2, 3]); // 1 2 3

// 문자열은 이터러블이다.
console.log(...'Hello'); // H e l l o

// Map,Set은 이터러블이다.
console.log(
  ...new Map([
    ['a', '1'],
    ['b', '2']
  ]);
); // ['a', '1'] ['b', '2']
```

spread 문법의 결과는 값이 아니다. 즉, spread 문법(`...`)이 피연산자를 연산하여 값을 생성하는 연산자가 아님을 의미한다. 따라서 spread 문법의 결과는 변수에 할당할 수 없다.

```javascript
const list = ...[1,2,3]; // Syntax Error: Unexpected token ...
```

## 함수 호출문의 인수 목록에서 사용하는 경우

```javascript
const arr = [1, 2, 3];

const max = Math.max(arr); // → NaN
```

`Math.max` 메서드는 매개변수 개수를 확정할 수 없는 가변 인자 함수다. 다음과 같이 개수가 정해져 있지 않은 여러 개의 숫자를 인수로 전달받아 인수 중에서 최대값을 반환한다.

```javascript
Math.max(1); // → 1
Math.max(1, 2); // → 2
Math.max(1, 2, 3); // → 3
Math.max(); // → Infinity
```

만약 `Math.max` 메서드에 숫자가 아닌 배열을 인수로 전달하면 최대값을 구할 수 없으므로 `NaN`을 반환한다.

```javascript
Math.max([1, 2, 3]); // → NaN
```

spread 문법이 존재하기 이전에는 배열을 펼쳐 요소들의 목록을 함수의 인수로 전달하고 싶은 경우 `Function.prototype.apply`를 사용하였다.

```javascript
var arr = [1, 2, 3];

// apply 함수의 2번째 인수(배열)는 apply 함수가 호출하는 함수의 인수 목록이다.
// 따라서 배열이 펼쳐져서 인수로 전달되는 효과가 있다.
var max = Math.max.apply(null, arr); // → 3
```

spread 문법을 사용하면 간결하고 가독성을 높이면서 사용할 수 있다.

```javascript
const arr = [1, 2, 3];

const max = Math.max(...arr);
```

spread 문법과 rest 파라미터는 형태는 동일하지만, 반대의 개념이다. rest 파라미터는 함수에 전달된 인수들의 목록을 배열로 전달받기 위해 사용하고, spread 문법은 여러 개의 값이 하나로 뭉쳐 있는 배열과 같은 이터러블을 펼쳐서 개별적인 값들의 목록을 만드는 것이다.

```javascript
function foo(...rest) {
  console.log(rest); // 1, 2, 3 → [1,2,3]
}

foo(...[1, 2, 3]); // [1,2,3] → 1, 2, 3
```

## 배열 리터럴 내부에서 사용하는 경우

spread 문법을 배열 리터럴에서 사용하면 ES5에서 사용하던 방식보다 간결하게 가독성을 높일 수 있다.

### concat

```javascript
// ES5
var arr = [1, 2].concat([3, 4]);
console.log(arr); // [1,2,3,4]

// ES6
const arr = [...[1, 2], ...[3, 4]];
console.log(arr); // [1,2,3,4]
```

### splice

ES5에서 어떤 배열의 중간에 다른 배열의 요소들을 추가하거나 제거하려면 `splice` 메서드를 사용한다. 이때 세 번째 인수로 배열을 전달하면 배열 자체가 추가된다.

```javascript
// ES5
var arr1 = [1, 4];
var arr2 = [2, 3];

// 세 번째 인수 arr2를 해체하여 전달하지 않으면 배열 자체가 추가된다.
arr1.splice(1, 0, arr2);
console.log(arr1); // [1, [2, 3], 4]

/*
  apply 메서드의 두 번째 인수는 apply 메서드가 호출한 splice 메서드의 인수 목록이다.
  apply 메서드의 두 번째 인수 [1, 0].concat(arr2)는 [1,0,2,3]으로 평가된다.
  따라서 splice 메서드에 apply 메서드의 두 번째 인수 [1,0,2,3]이 해체되어 전달된다.
*/
Array.prototype.splice.apply(arr1, [1, 0].concat(arr2));
console.log(arr1); // [1,2,3,4]
```

spread 문법을 사용하면 다음과 같이 더욱 간결하고 가독성 좋게 표현할 수 있다.

```javascript
// ES6
const arr1 = [1, 4];
const arr2 = [2, 3];

arr1.splice(1, 0, ...arr2);
console.log(arr1); // [1,2,3,4]
```

### slice

ES5에서는 배열을 복사하기 위해 `slice` 메서드를 사용하였으나 ES6부터는 spread 문법을 이용하면 가독성을 높일 수 있다.

```javascript
// ES5
var origin = [1, 2];
var copy = origin.slice();

console.log(copy); // [1,2]
console.log(copy === origin); // false

// ES6
const origin = [1, 2];
const copy = [...origin];

console.log(copy); // [1,2]
console.log(copy === origin); // false
```

이때 `slice`나 spread 문법 둘 다 원본 배열의 각 요소를 얕은 복사(Shallow copy)하여 새로운 복사본을 생성한다.

### 이터러블을 배열로 변환

ES5에서 이터러블을 배열로 변환하려면 `Function.prototype.apply` 또는 `Function.prototype.call` 메서드를 사용하여 `slice` 메서드를 호출해야 한다.

```javascript
// ES5
function sum() {
  // 이터러블이면서 유사 배열 객체인 arguments를 배열로 변환
  var args = Array.prototype.slice.call(arguments);

  return args.reduce(function (pre, cur) {
    return pre + cur;
  }, 0);
}
console.log(sum(1, 2, 3)); // 6
```

이 방법은 이터러블 뿐만 아니라 이터러블이 아닌 유사 배열 객체도 배열로 변환할 수 있다.

```javascript
// 이터러블이 아닌 유사 배열 객체
const arrayLike = {
  0: 1,
  1: 2,
  2: 3,
  length: 3
};

const arr = Array.prototype.slice.call(arrayLike); // → [1, 2, 3]
console.log(Array.isArray(arr)); // true
```

spread 문법을 사용하면 좀 더 간편하게 이터러블을 배열로 변환할 수 있다. `arguments` 객체는 이터러블이면서 유사 배열 객체이다. 따라서 스프레드 문법의 대상이 될 수 있다.

```javascript
function sum() {
  return [...arguments].reduce((pre, cur) => pre + cur, 0);
}
console.log(sum(1, 2, 3)); // 6
```

또한 rest 파라미터를 사용하는 방식도 가능하다.

```javascript
const sum = (...args) => args.reduce((pre, cur) => pre + cur, 0);
console.log(sum(1, 2, 3)); // 6
```

단, 이터러블이 아닌 유사 배열 객체는 스프레드 문법의 대상이 될 수 없다.

```javascript
const arrayLike = {
  0: 1,
  1: 2,
  2: 3,
  length: 3
};
const arr = [...arrayLike];
// TypeError: object is not iterable (cannot read property Symbol(Symbol.iterator))
```

이터러블이 아닌 유사 배열 객체를 배열로 변경하려면 ES6에서 도입된 `Array.from` 메서드를 사용한다. `Array.from` 메서드는 유사 배열 객체 또는 이터러블을 인수로 전달받아 배열로 변환하여 반환한다.

```javascript
Array.from(arrayLike); // → [1,2,3]
```

### 객체 리터럴 내부에서 사용하는 경우

2020년 7월 기준 TC39 프로세서의 stage 4(Finished) 단계에 제안되어 있는 spread 프로퍼티를 사용하면 객체 리터럴의 프로퍼티 목록에서도 spread 문법을 사용할 수 있다. spread 문법의 대상은 이터러블이었지만 spread 프로퍼티 제안에서는 일반 객체를 대상으로도 spread 문법의 사용을 허용한다.

```javascript
// spread 프로퍼티

// 객체 복사(얕은 복사)
const obj = { x: 1, y: 2 };
const copy = { ...obj };
console.log(copy); // { x: 1, y: 2 }
console.log(obj === copy); // false

// 객체 병합
const merged = { x: 1, y: 2, ...{ a: 3, b: 4 }
console.log(merged); // { x: 1, y: 2, a: 3, b: 4 }
```

spread 프로퍼티가 제안되기 이전에는 ES6에서 도입된 `Object.assign` 메서드를 사용하여 병합하거나 프로퍼티를 변경 혹은 추가하였다.

```javascript
// 객체 병합. 프로퍼티 중복 시 뒤에 위치한 프로퍼티가 우선된다.
const merged = Object.assign({}, { x: 1, y: 2 }, { y: 10, z: 3 });
console.log(merged); // { x: 1, y: 10, z: 3 }

// 특정 프로퍼티 변경
const changed = Object.assign({}, { x: 1, y: 2 }, { y: 100 });
console.log(changed); // { x: 1, y: 100}

// 프로퍼티 추가
const added = Object.assign({}, { x: 1, y: 2 }, { z: 0 });
console.log(added); // { x: 1, y: 2, z: 0 }
```

spread 프로퍼티는 `Object.assign` 메서드를 대체할 수 있는 간편한 문법이다.

```javascript
const merged = { ...{ x: 1, y: 2 }, ...{ y: 10, z: 3 } };
console.log(merged); // { x: 1, y: 10, z: 3 }

const changed = { ...{ x: 1, y: 2 }, y: 100 };
console.log(chaged); // { x: 1, y: 100 }

const added = { ...{ x: 1, y: 2 }, z: 0 };
console.log(added); // { x: 1, y: 2, z: 0 }
```

## Reference

- [모던 자바스크립트 Deep Dive 35장 - 스프레드 문법](http://www.yes24.com/Product/Goods/92742567)
