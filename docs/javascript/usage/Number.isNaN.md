# Number.isNaN()

```javascript
let a = 2 / 'foo';
typeof a === 'number'; // true
```

어떠한 변수가 `NaN`인지 확인할 때 주의해야 할 점은 `NaN`은 심지어 자기 자신과도 같지 않다는 점이다.

```javascript
a == NaN; // false
a === NaN; // false
```

ES6 이전까지는 전역 함수인 `isNaN`을 이용하여 `NaN` 여부를 확인하곤 했다. 그러나 이 함수는 실제로 '인자 값이 숫자인지 여부를 평가'하는 기능이 전부인 함수이다.

```javascript
let a = 2 / 'foo';
let b = 'foo';

a; // NaN
b; // "foo"

isNaN(a); // true
isNaN(b); // true(?)
```

ES6에서 도입된 `Number.isNaN` 정적 메서드는 인수로 전달된 숫자값이 `NaN`인지 검사하여 그 결과를 반환한다.

```javascript
// 인수가 NaN이면 true를 반환한다.
Number.isNaN(NaN); // true
```

Built-in 전역 함수 `isNaN`과는 차이가 있는데, 전달받은 인수를 숫자로 암묵적 타입 변환을 진행하지 않는다.따라서 숫자가 아닌 인수가 주어졌을 때 반환값은 언제나 `false`이다.

```javascript
// Number.isNaN은 인수를 숫자로 암묵적 타입 변환하지 않는다.
Number.isNaN(undefined); // → false

// isNaN은 인수를 숫자로 암묵적 타입 변환한다. undefined는 NaN으로 암묵적 타입 변환된다.
isNaN(undefined); // → true
```

또한 `NaN`이 자기 자신과 동등하지 않다는 성질을 이용하여 Polyfill을 구현하는 것도 가능하다.

```javascript
if (!Number.isNaN) {
  Number.isNaN = function (n) {
    return n !== n;
  };
}
```

ES6에 도입된 `Object.is()` 메서드를 이용하여도 `NaN`을 식별할 수 있다.

```javascript
Object.is(NaN, NaN); // true
```

[문자열 다루기 기본](https://programmers.co.kr/learn/courses/30/lessons/12918) 문제를 풀던 도중 `Number.isNaN`을 활용하여 문제를 풀어보고자 시도하던 도중 생긴 궁금증 때문에 정리하게 되었다. 문제가 조금 명확하지 않은 듯 하지만 `1e12` 꼴의 지수 형태의 input을 처리하는 것이 문제였던 것 같다.

## Reference

- [You don't know js - 2장 값 - 특수 숫자](http://www.yes24.com/Product/Goods/43219481?OzSrank=6)
- [모던 자바스크립트 Deep Dive 28장 - Number 메서드](http://www.yes24.com/Product/Goods/92742567)
