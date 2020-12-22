# 집합(Set) 구현하기

중복 요소가 제거된 배열을 만드는 방법에는 다양한 방법이 존재한다.

```javascript
const arr = [1, 1, 2, 2, 3, 4];

console.log([new Set(arr)]); // [1,2,3,4]

arr.reduce((ret, v) => {
  if (!ret.includes(v)) {
    ret = [...ret, v];
  }
  return ret;
}, []); // [1,2,3,4]

arr.reduce((ret, v) => {
  if (ret.indexOf(v) === -1) {
    ret = [...ret, v];
  }
  return ret;
}, []); // [1,2,3,4]
```

그러나 배열의 요소가 객체 타입의 값인 경우 위의 방법으로는 중복 요소를 제거할 수 없다.

```javascript
const arr = [
  [1, 1],
  [1, 1],
  [1, 1],
  [2, 3]
];

const set = [...new Set(arr)];
set; // [[1,1],[1,1],[1,1],[2,3]]

arr.reduce((ret, v) => {
  if (!ret.includes(v)) {
    ret = [...ret, v];
  }
  return ret;
}, []); // [[1,1],[1,1],[1,1],[2,3]]

arr.reduce((ret, v) => {
  if (ret.indexOf(v) === -1) {
    ret = [...ret, v];
  }
  return ret;
}, []); // [[1,1],[1,1],[1,1],[2,3]]
```

원인을 파악하기 위해 `includes()` 메서드나 `Set`이 중복 요소를 판별할 때 어떤 방식을 이용하는 지 확인해보자. 먼저 ES6에는 4가지의 동치 비교 알고리즘이 존재한다.

- Abstract Equality Comparison (`==`)
- Strict Equality Comparison (`===`)
  - `Array.prototype.indexOf`, `Array.prototype.lastIndexOf`, `case` 매칭에 사용된다.
- SameValueZero
  - `Map`,`Set`,`TypedArray`,`ArrayBuffer`,`String.prototype.includes`에 쓰인다.
- SameValue
  - 그 외 모든 곳에서 사용된다.

?> SameValueZero, SameValue 알고리즘을 살펴보면 기본적으로 Strict Equality Comparison(`===`)과 유사하지만 0을 처리하는 방식과 `NaN`을 처리하는 방식에 있어서만 차이가 난다. 이 중에서 SameValue 알고리즘은 `Object.is()` 메서드로 제공된다.

각각의 처리 방식에는 미묘한 차이가 존재하지만 우선 핵심적인 사항은 다음과 같다.

```javascript
/* Primitive values */
1 === 1; // true
Object.is(NaN, NaN); // true

/* Object values */
[1,2] === [1,2]; // false
{} === {}; // false
```

위의 코드에서 사람이 인식하기에는 `[1,2]`와 `[1,2]`는 형태와 내용이 동일하기 때문에 같은 배열로 인식하고 싶어한다. 하지만 엔진이 인식하는 바로는 해당 코드가 실행될 때 리터럴이 평가되어 새로운 객체가 생성되므로 각각의 객체를 가리키는 참조값이 다르기 때문에 다르다고 인식하는 것이다.

그렇다면 객체 타입의 값이 배열의 요소로 설정되어 있을 때 기존의 `includes()`,`indexOf()`,`Set` 등으로 중복 요소를 제거하는 로직은 사용하기 힘들어 보인다. 그렇다면 배열의 요소가 배열인 경우와 객체인 경우를 분리해서 생각해보자.

!> 모든 경우를 cover할 수 있는 로직을 구성하는 것이 아니라 알고리즘 문제에서 자주 등장하는 경우에 대해서만 처리할 수 있게끔 구현한다. 기본적으로 배열의 모든 요소의 타입은 동일하다 가정한다.

## 배열의 요소가 다시 배열인 경우

배열의 요소가 다시 배열인 경우에는 찾고자 하는 배열이 전체 배열 내에 존재해야 하는지 먼저 확인해야 한다. 그러기 위해서는 두 배열의 내용이 동일한 지 판별할 수 있어야 한다.

```javascript
const arrayEquals = (a, b) => {
  return (
    Array.isArray(a) &&
    Array.isArray(b) &&
    a.length === b.length &&
    a.every((aElement, aIdx) => aElement === b[aIdx])
  );
};
```

먼저 매개변수로 전달받은 `a`,`b`가 모두 배열인지 확인한다. `a`,`b`가 둘 다 배열이라면 `length` 프로퍼티를 가질 것이고, 또 두 배열의 길이가 다르다면 다른 배열이므로 확인한다. 마지막으로는 `a`의 모든 요소가 `b`의 동일한 위치에 있는 요소와 같은지 확인하면 된다.

그 다음으로는 전체 배열에 해당 배열이 존재하는 지 확인하는 로직이 필요하다.

```javascript
const isIncludes = (src, target) => {
  return src.some(element => {
    return arrayEquals(element, target);
  });
};
```

전체 배열의 한 요소라도 확인하고자 하는 배열과 동일하다고 판별되면 전체 배열이 해당 요소를 포함하고 있다는 의미이므로 위와 같이 구현 가능할 것이다.

```javascript
const arr = [
  [1, 1],
  [1, 1],
  [1, 1],
  [2, 3]
];

arr.reduce((src, target) => {
  return isIncludes(src, target) ? src : [...src, target];
}, []); // [[1,1],[2,3]]
```

## 배열의 요소가 객체인 경우

배열의 요소가 객체인 경우는 다소 복잡하다.

!> 일반적으로 알고리즘 문제에서 등장하는 경우와 동일하게 프로퍼티 값은 다시 중첩 객체를 가질 수 있으나, 그 값으로는 원시 값이나 배열만 갖는다고 가정한다.

```javascript
const objectEquals = (a, b) => {
  if (Object.is(a, b)) return true;
  const aKeys = Object.keys(a);
  const bKeys = Object.keys(b);
  if (aKeys.length !== bKeys.length) return false;
  for (const key of aKeys) {
    if (
      typeof a[key] === 'object' &&
      typeof b[key] === 'object' &&
      a[key] !== null &&
      b[key] !== null
    ) {
      if (!objectEquals(a[key], b[key])) return false;
    } else {
      if (a[key] !== b[key]) return false;
    }
  }
  return true;
};
```

일단 `a`와 `b`가 둘 다 `NaN`인 경우를 처리하기 위해 `Object.is()` 메서드를 이용하여 비교한다. 그 다음 `a`,`b` 객체의 프로퍼티 키들을 배열로 만들어 주는 `Object.keys()` 메서드를 이용하여 프로퍼티 키의 갯수를 비교한다. 그 다음 각각의 프로퍼티 키가 갖고 있는 값들 간의 비교를 진행하게 되는데, 프로퍼티 값이 중첩 객체인 경우 재귀 호출을 이용하여 탐색을 진행하게 된다.

!> 단, `null` 역시 `typeof` 연산자를 이용하면 `object`로 인식되므로 별도의 예외 처리가 필요하다.

동일한지 비교할 대상이 객체인지, 배열인지에 따라 별도의 함수를 구현하기 보다는 기존의 함수의 매개변수로 비교 함수를 넘겨주는 방식으로 수정해주면 보다 유연한 구조를 가질 수 있을 것이다.

```javascript
const isIncludes = (src, target, cb) => {
  return src.some(element => {
    return cb(element, target);
  });
};
```

```javascript
const objArr = [
  { start: [1, 2], end: [3, 4] },
  { start: [1, 2], end: [3, 4] },
  { start: [3, 4], end: [5, 6] }
];

objArr.reduce((src, target) => {
  return isIncludes(src, target, objectEquals) ? src : [...src, target];
}, []);

/* 
  [
    { start: [1, 2], end: [3, 4] },
    { start: [3, 4], end: [5, 6] }
  ]
*/
```
