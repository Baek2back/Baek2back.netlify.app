# 함수형 자바스크립트

> **ES5에 도입된 배열 고차 함수**

1. `Array.prototype.map`
2. `Array.prototype.filter`
3. `Array.prototype.reduce`
4. `...etc`

위의 고차 함수들은 기본적으로 `Array.prototype`에 정의되어 있기 때문에 `Array-Like` 객체나 `object({})`에는 적용할 수 없다.

> **`Array-Like` 객체**

배열과 유사하지만 배열은 아닌 객체를 의미하며 다음과 같은 특징을 가진다.

1. `length` 프로퍼티를 가진다.
2. `index`로 접근 가능하다.
3. `Symbol.iterator` 프로퍼티를 가지고 있으므로 이터러블이다.
4. `Array.prototype`을 상속받지 않는다.

?> 기존의 배열 고차 함수는 유사 배열 객체가 아닌 순수 배열에만 적용 가능하지만, 유사 배열 객체(이터러블)에도 적용 가능한 함수를 만들고, 함수들을 조합해 결과를 도출해보자.

우선은 다음과 같은 데이터가 존재한다 생각해보자.

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 }
];
```

기본적으로 이터러블을 순회하는 `for...of` 문은 다음과 동일하다.

```javascript
for (const a of iter) {
  ...
}
// for...of는 다음과 동일하다.
iter = iter[Symbol.iterator]();
let cur;
while (!((cur = iter.next()).done)) {
  const a = cur.value;
  ...
}
```

## map

```javascript
const map = (f, iter) => {
  const res = [];
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    res.push(f(a));
  }
  return res;
};

map(p => p.price, products);
// (5) [15000, 20000, 15000, 30000, 25000]
map(p => p.name, products);
// (5) ['반팔티', '긴팔티', '핸드폰케이스', '후드티', '바지]
```

## 이터러블인 요소면 모두 적용 가능한 map의 다형성

```javascript
const str = 'abc';
const nodeList = document.querySelectorAll('*');
// NodeList(5) [html, head, script, script, script]
nodeList.map(el => el.nodeName);
// Uncaught TypeError: ...map is not a function
nodeList[Symbol.iterator]();
// Array Iterator {}
map(el => el.nodeName, nodeList);
// (5) ['html', 'head', 'script', 'script', 'script']

str[Symbol.iterator]();
// String Iterator {}
map(v => v, str);
// (3) ['a', 'b', 'c']

let m = new Map();
m.set('a', 10);
m.set('b', 20);
m[Symbol.iterator]();
// Map Iterator {'a' => 10, 'b' => 20}
m.map(([k, v]) => [k, v * 2]);
// Uncaught TypeError: m.map is not a function
map(([k, v]) => [k, v * 2], m);
// (2) [Array(2), Array(2)]

const obj = {};
obj[Symbol.iterator]();
// Uncaught TypeError: obj[Symbol.iterartor] is not a function
```

이처럼 순수 배열에만 적용 가능한 `Array.prototype.map`과 달리 직접 구현한 `map`은 이터러블 프로토콜을 따른다면 유사 배열에도 적용하는 것이 가능하다.

## filter

```javascript
const filter = (f, iter) => {
  const res = [];
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    if (f(a)) res.push(a);
  }
  return res;
};
```

## reduce

```javascript
// reduce의 동작 방식
const add = (a, b) => a + b;
reduce(add, 0, [1, 2, 3, 4, 5]); // 15
// 다음과 같다.
add(add(add(add(add(0, 1), 2), 3), 4), 5); // 15

const reduce = (f, acc, iter) => {
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    acc = f(acc, a);
  }
  return acc;
};

// 초기값을 누락한 경우에는 이터러블의 첫 번째 요소를 사용하도록 리팩토링
reduce(add, [1, 2, 3, 4, 5]);
reduce(add, 1, [2, 3, 4, 5]);

const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  // iter: [2,3,4,5]
  // acc: 1
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    acc = f(acc, a);
  }
  return acc;
};

log(
  reduce(
    (totalPrice, product) => totalPrice + product,
    0,
    map(product => product.price, products)
  )
); // 105000
log(
  reduce(
    (totalPrice, product) => totalPrice + product,
    map(product => product.price, products)
  )
); // 105000
```

## map,filter,reduce 활용

```javascript
const add = (a, b) => a + b;
reduce(
  add,
  map(
    p => p.price,
    filter(p => p.price < 20000, products)
  )
); // 30000

reduce(
  add,
  filter(
    price => price < 20000,
    map(p => p.price, products)
  )
); // 30000
```

## go

```javascript
// go는 즉시 평가하는 데 사용된다.
go(
  0,
  a => a + 1,
  a => a + 10,
  a => a + 100,
  log
); // 111

// 위와 같이 실행되도록 하기 위해 우선 인수들이 배열이라고 생각해보자.
go([0, a => a + 1, a => a + 10, a => a + 100, console.log]);

const go = list => {
  log(list);
}; // (5) [0, f, f, f ,f]

// 그렇지만 우선은 인자에 배열을 넘긴다는 제약이 없으므로, 다음과 같이 바꿀 수 있을 것이다.

const go = (...list) => {
  log(list);
}; // (5) [0, f, f, f, f]

const go = (...args) => {
  return reduce((arg, f) => f(arg), args);
};

go(
  products,
  products => filter(p => p.price < 20000, products),
  products => map(p => p.price, products),
  prices => reduce(add, prices),
  log
); // 30000
```

## pipe

```javascript
// pipe는 함수를 합성하는 데 사용한다. 즉, 함수를 리턴하는 함수이다.
const f = pipe(
  a => a + 1,
  a => a + 10,
  a => a + 100
);
log(f(0)); // 111

const pipe = (...fs) => arg => {
  return go(arg, ...fs);
};

// go의 경우에는 인자에 함수가 들어와도 제대로 동작한다.
go(
  add(0, 1),
  a => a + 1,
  a => a + 10,
  a => a + 100,
  log
); // 112

// pipe의 경우는 다음과 같이 사용해야 하는 제약이 있다.
log(f(add(0, 1)));

// 다음과 같이 사용할 수 있도록 수정해보자.
const f = pipe(
  add,
  a => a + 1,
  a => a + 10,
  a => a + 100
);
log(f(0, 1)); // 112

const pipe = (f, ...fs) => (...args) => {
  return go(f(...args), ...fs);
};
```

## curry

> **커링 함수**(currying function)란 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 함수를 의미한다. 커링은 한 번에 하나의 인자만 전달하는 것을 원칙으로 하며, 중간 과정에서 함수를 실행한 결과는 그 다음 인자를 받기 위해 대기할 뿐이므로, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않는다(부분 적용 함수는 여러 개의 인자를 전달할 수 있고, 실행 결과를 재실행할 때 원본 함수가 무조건 실행된다).

```javascript
/**
  1. 함수를 받아서 함수를 리턴한다.
  2. 그렇게 리턴된 함수가 호출되었을 때
    (1) 인자가 2개 이상이라면 리턴했던 함수를 즉시 실행한다.
    (2) 인자가 2개 미만이라면 함수를 다시 리턴한 뒤 나중에 받은 인자들을 합쳐서 실행한다.
*/
const curry = f => (a, ..._) => (_.length ? f(a, ..._) : (..._) => f(a, ..._));

const mult = curry((a, b) => a * b);
log(mult(2)); // (..._) => f(a, ..._)
log(mult(2)(3)); // 6

const map = curry((f, iter) => {
  const res = [];
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    res.push(f(a));
  }
  return res;
});

const filter = curry((f, iter) => {
  const res = [];
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    if (f(a)) res.push(a);
  }
  return res;
});

const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    acc = f(acc, a);
  }
  return acc;
});

// curry 적용 이전
go(
  products,
  products => filter(p => p.price < 20000, products),
  products => map(p => p.price, products),
  prices => reduce(add, prices),
  log
); // 30000

// curry 적용 이후
go(
  products,
  products => filter(p => p.price < 20000)(products),
  products => map(p => p.price)(products),
  products => reduce(add)(products),
  log
);

// Point-Free style
go(
  products,
  filter(p => p.price < 20000),
  map(p => p.price),
  reduce(add),
  log
); // 30000
```

## range

```javascript
log(range(5));
// 위의 코드의 출력 결과를 [0,1,2,3,4]로 만들어보자.

const range = length => {
  let i = -1;
  const res = [];
  while (++i < length) res.push(i);
  return res;
};

let list = range(5);
log(list); // [0, 1, 2, 3, 4]
log(reduce(add, list)); // 10

const L = {};
L.range = function* (length) {
  let i = -1;
  while (++i < length) yield i;
};

let list = L.range(5);
log(list);
// GeneratorFunctionPrototype {...}
log(reduce(add, list)); // 10

/*
  range, L.range의 가장 큰 차이는 각각 생성해 낸 list의 내용이 있다.
  우선 range의 경우 전체 배열이 출력되고, L.range의 경우 이터레이터가 출력된다.
  하지만 최종적으로 두 함수 모두 같은 결과를 만들어 낸 이유는 reduce가 이터러블을 인자로 받기 때문이다.
  배열과 이터레이터 모두 이터러블이지만 둘의 차이는 이터레이터는 next()가 실행되기 전에는 평가되지 않는다는 점이다.
  따라서 L.range의 결과를 얻는데 있어서 미리 [0,1,2,3,4,5]라는 배열을 생성해 둘 필요가 없는 것이다.
*/
```

## take

```javascript
log(take(2, [1, 2, 3, 4, 5]));
// 전체 이터러블에서 2개의 요소만을 선택하게끔 구현해보자.
// [1, 2]

const take = (limit, iter) => {
  const res = [];
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    res.push(a);
    if (limit === res.length) return res;
  }
  return res;
};

log(take(10, range(Infinity)));
// range는 전체 배열을 미리 생성하기 때문에 RangeError가 발생한다.
log(take(10, L.range(Infinity)));
// L.range의 경우 필요할 때 값을 평가하기 때문에 문제없이 동작한다.

const take = curry(...);

go(
  range(10000),
  take(5),
  reduce(add),
  log
);
// [0,1,2,3,4, ..., 9999]
// [0,1,2,3,4]
// 10

go(
  L.range(10000),
  take(5),
  reduce(add),
  log
);
// [0,1,2,3,4]
// 10
```

## L.map, L.filter

```javascript
const L = {};
L.map = function* (f, iter) {
  for (const a of iter) {
    yield f(a);
  }
};

let it = L.map(a => a + 10, [1, 2, 3]);
log(it);
// GeneratorFunctionPrototype {...}
log([...it]);
// [11,12,13]
// 위의 코드는 다음과 같다.
log(it.next().value);
// 11
log(it.next().value);
// 12
log(it.next().value);
// 13

L.filter = function* (f, iter) {
  for (const a of iter) {
    if (f(a)) yield a;
  }
};
let it = L.filter(a => a % 2, [1, 2, 3]);
log(it);
// GeneratorFunctionPrototype {...}
log([...it]);
// [1,3]

L.map = curry(...);
L.filter = curry(...);
```

## 제때 계산법 vs 느긋한 계산법

```javascript
// (1) range,map,filter,take,reduce
go(
  range(10),
  map(n => n + 10),
  filter(n => n % 2),
  take(2),
  reduce(add),
  log
);
/*
  [0,1,2,3,4,5,6,7,8,9],
  [10,11,12,13,14,15,...],
  [11,13,15,17,19],
  [11,13],
  24
*/

// (2) L.range,L.map,L.filter,take,reduce
go(
  L.range(10),
  L.map(n => n + 10),
  L.filter(n => n % 2),
  take(2),
  reduce(add),
  log
);
/*
  [    0  ][   1   ][    2  ][    3   ]
  [   10  ][  11   ][   12  ][   13   ]
  [ false ][ true  ][ false ][  true  ]
  [ ----- ][  11(✓)][ ----- ][   13(✓)]
  24
*/

/*
  두 방식 모두 결과는 동일하지만, 가장 큰 차이는 (1)의 경우 위에서부터 모두 평가하면서 내려가지만, (2)의 경우 평가를 미룬 이터레이터를 위에서부터 반환한다는 점이다.
  (1) range → map → filter → take → reduce → log
  (2) reduce → take → L.filter → L.map → L.range(0) → L.map(10) → L.filter(10 % 2 → false) → take(제외) → ...
  따라서 (2)의 경우 take할 요소를 filter에게 요청하고, filter는 다시 map에게 필터링할 요소를 요청하는 방식으로 진행되는 것이다.
*/
```

## find

```javascript
const users = [
  { age: 32 },
  { age: 29 },
  { age: 28 },
  { age: 37 },
  { age: 15 },
  { age: 22 }
];

log(find(u => u.age < 30, users));
// Object {age: 29}

const find = (f, iter) => go(iter, filter(f), take(1), ([a]) => a);

// 위의 find 함수에서 아쉬운 점은 한 개의 결과만을 필요로 하는 데도 불구하고 모두 순회한다는 점이다.
// 따라서 filter를 L.filter로 변경하여 필요한 값을 얻을 때 까지만 순회하는 방식으로 바꾸면 더 효율적이다.
const find = (f, iter) => go(iter, L.filter(f), take(1), ([a]) => a);
```

## L.map, L.filter를 이용하여 map,filter 구현하기

```javascript
const takeAll = take(Infinity);

L.map = curry(function* (f, iter) {
  for (const a of iter) {
    yield f(a);
  }
});

const map = curry((f, iter) => go(iter, L.map(f), takeAll));
const map = curry(pipe(L.map, takeAll));

L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    if (f(a)) yield a;
  }
});

const filter = curry((f, iter) => go(iter, L.filter(f), takeAll));
const filter = curry(pipe(L.filter, takeAll));
```

## L.flatten, flatten

```javascript
log([[1,2],3,4,[5,6],[7,8,9]]);
// [Array[2]],3,4,Array[2],Array[3]]
log([...[1,2],3,4,...[5,6],...[7,8,9]]);
// [1,2,3,4,5,6,7,8,9]

const isIterable = (a) => a && a[Symbol.iterator];

L.flatten = function* (iter) {
	for (const a of iter) {
		if (isIterable(a)) for (const b of a) yield b;
		else yield a;
};

L.flatten = function* (iter) {
	for (const a of iter) {
		if (isIterable(a)) yield* a
		else yield a;
	}
};

log([...L.flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]])]);
// [1,2,3,4,5,6,7,8,9]

const flatten = pipe(L.flatten, takeAll);
```

## L.deepFlat

```javascript
L.deepFlat = function* f(iter) {
  for (const a of iter) {
    if (isIterable(a)) yield* f(a);
    else yield a;
  }
};

log([...L.deepFlat([1, [2, [3, 4], [[5]]]])]);
// [1,2,3,4,5]
```

## L.flatMap

```javascript
log(
  [
    [1, 2],
    [3, 4],
    [5, 6, 7]
  ].flatMap(a => a.map(a => a + a))
);
// [2,4,6,8,10,12,14]
log(
  flatten(
    [
      [1, 2],
      [3, 4],
      [5, 6, 7]
    ].map(a => a.map(a => a + a))
  )
);
// [2,4,6,8,10,12,14]

L.flatMap = curry(pipe(L.map, L.flatten));
```

## callback vs Promise

기본적으로 callback과 `Promise`의 가장 큰 차이는 `Promise`의 경우 `Promise` 객체를 반환한다는 점이다.

```javascript
function add10(a, callback) {
  setTimeout(() => callback(a + 10), 100);
}

function add20(a) {
  return new Promise(resolve => setTimeout(() => resolve(a + 20), 100));
}

const callbackResult = add10(5, log);
const promiseResult = add20(5);

callbackResult; // undefined
promiseResult; // Promise {}
```

따라서 `Promise`의 경우 비동기 상황을 값으로 취급하기 때문에 상태를 검사할 수 있게 되고, 재사용이 가능해진다는 특징을 갖는다.

## 값으로서의 Promise 활용

```javascript
const go1 = (a, f) => f(a);
const add5 = a => a + 5;
go1(10, add5); // 15
```

위와 같은 코드가 잘 동작하기 위해서 우선 `go1` 함수에 인수로 전달된 함수 `f`가 동기 방식으로 동작해야 하며, `a` 역시 `f` 함수를 호출할 당시에는 `Promise`가 아닌 값으로 평가되어야 정상적으로 동작하게 될 것이다.

```javascript
const delay100 = a => new Promise(resolve => setTimeout(() => resolve(a), 100));

const go1 = (a, f) => f(a);
const add5 = a => a + 5;

go1(10, add5); // 15
go1(delay100(10), add5); // [object Promise]5
```

만약 `a`에 전달된 값이 `Promise`라면 `f`를 호출할 당시의 값은 `Promise`가 되어 의도대로 동작하지 않을 것이다.

```javascript
const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));
go1(10, add5); // 15
go1(delay100(10), add5); // Promise {<pending>}
```

`go1` 함수를 수정하여 `go1`에 전달된 첫 번째 인자가 `Promise`인 경우에는 해당 `Promise`에 두 번째 인자로 전달받은 함수를 넘겨주는 방식으로 수정하면 잘 동작한다. 또한 `Promise` 객체가 반환되기 때문에 `then()` 메서드를 이용하여 추가적인 작업을 지속적으로 수행할 수 있게 된다.

## 함수 합성 관점에서의 Promise와 모나드

### 모나드

함수 합성을 안전하게 수행할 수 있게 도와주는 값을 감싸고 있는 일종의 박스(`[1]`)

```javascript
const f = a => a * a;
const g = a => a + 1;

f(g(1)); // 4
f(g()); // NaN
```

함수 `f`와 `g`를 합성한다 했을 때, 일반적인 숫자 값이 인수로 제공된다면 문제 없겠지만 자바스크립트의 특성 상 매개변수의 타입을 강제하거나 확인할 방법이 없기 때문에 안전하게 합성했다고 할 수 없다. 따라서 함수 합성을 안전하게 하기 위해 등장한 것이 **모나드**라는 개념이다.

자바스크립트의 빌트인 객체 `Array` 역시 모나드라 할 수 있다.

```javascript
[1]
  .map(g)
  .map(f)
  .forEach(v => console.log(v)); // 4
[]
  .map(g)
  .map(f)
  .forEach(v => console.log(v));
```

배열 내부에 아무런 값이 존재하지 않더라도 에러나 의도치 않은 동작이 발생하지 않는 것을 확인할 수 있다.

### Promise

```javascript
new Promise(resolve => setTimeout(() => resolve(2), 100))
  .then(g)
  .then(f)
  .then(v => console.log(v));
```

`Promise`는 비동기적 상황에 대해서 안전하게 함수 합성을 하기 위한 도구로, 비동기적으로 생성되는 값들에 대해 안전하게 핸들링 하기 위한 도구라 할 수 있다.

## Kleisli Composition 관점에서의 Promise

Kleisli Composition은 오류가 발생할 수 있는 상황에서 함수 합성을 안전하게 수행하는 규칙이며, 실제 프로그래밍 영역에서는 외부에서의 상태 변경 등으로 인해 `f(g(x) === f(g(x))`가 항상 성립하지 않는 상황이 발생할 수 있다.

```javascript
const users = [
  { id: 1, name: 'a' },
  { id: 2, name: 'b' },
  { id: 3, name: 'c' }
];

const getUserById = id => find(u => u.id === id, users);
const f = ({ name }) => name;
const g = getUserById;
const f_g = id => f(g(id));

f_g(2) === f_g(2); // true
```

위와 같은 상황에서는 아무 문제가 발생하지 않겠지만, 찾고자 하는 `id`의 유저가 존재하지 않거나, 외부에서의 상태 조작으로 인해 기존에 존재하던 `id`의 유저가 없어지게 되는 경우에는 문제가 발생할 것이다.

```javascript
f_g(4); // Uncaught TypeError: Cannot destructure property 'name' of 'undefined'

users.pop();
f_g(3); // Uncaught TypeError: Cannot destructure property 'name' of 'undefined'
```

이러한 상황에서 문제가 발생하지 않도록 도와주는 것이 Kleisli Composition이다.이때 함수 합성을 `Promise`를 통해 수행하며 값이 존재하지 않는 경우 `Promise.reject()`를 반환하게끔 하면 된다. 이를 통해 도중에 `Promise.reject`가 반환되는 경우 이후의 함수 합성을 더 이상 진행하지 않고 `catch`로 이동하게 된다.

```javascript
const getUserById = id =>
  find(u => u.id === id, users) || Promise.reject('Not exist');

const f_g = id =>
  Promise.resolve(id)
    .then(g)
    .then(f)
    .catch(e => e);
f_g(2).then(console.log); // b

users.pop();
users.pop();

f_g(2).then(console.log); // Not exist
```

## go, pipe, reduce에서 비동기 제어

`Promise` 역시 비동기 상황을 값으로 취급하기 때문에 `go`,`pipe`,`reduce`에서도 대응하게끔 만들 수 있다. 또한 중간에 `reject`가 발생한 경우에도 대응할 수 있게끔 해야 한다.

```javascript
go(
  1,
  a => a + 10,
  a => a + 100,
  a => a + 1000,
  log
); // 1111

go(
  1,
  a => a + 10,
  a => Promise.resolve(a + 100),
  a => a + 1000,
  log
); // [object Promise]1000
```

기존의 구현에서는 `Promise`를 제대로 처리하지 못하는 것을 확인할 수 있다. 이는 `go` 함수 내부 구현의 `reduce` 때문이다. `pipe` 역시 `reduce`를 활용하므로 `reduce`를 바꿔주면 해결된다.

```javascript
const go = (...args) => reduce((a, f) => f(a), args);
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});
```

기존의 `reduce`에서는 `Promise`가 전달되면 `f(Promise{}, a)`와 같은 형태로 호출하게 되어 의도치 않게 동작하게 되는 것이다.

> **Solution 1**

```javascript
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = acc instanceof Promise ? acc.then(acc => f(acc, a)) : f(acc, a);
  }
  return acc;
});
```

첫 번째 방법은 동작은 하지만 함수 실행 도중 중간에 한 번이라도 `Promise`를 만나게 되면 그 이후에는 계속 *Promise Chaning*이 수행되게 되고, 함수 합성이 적은 경우에는 큰 차이가 없겠지만 많은 함수가 합성되어 있는 경우 하나의 콜 스택에서 실행되지 않게되어 성능 저하 역시 발생할 것이다. 즉, `Promise` 이후에 실행되는 함수에 대해서는 다시 동기적으로 평가하고 싶다면 다른 방식을 채택해야 할 것이다.

!> 콜 스택과 프로미스 체인에 대해 추후에 추가적으로 정리할 것!

> **solution 2**

```javascript
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  return (function recur(acc) {
    for (const a of iter) {
      acc = f(acc, a);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  })(acc);
});
```

재귀 함수(`recur`)를 이용해 `acc`가 `Promise` 객체인지 확인하고, `Promise` 객체인 경우에는 재귀적으로 호출하게 되어 `Promise` 객체가 갖고 있는 값이 전달되게 되어, 이후에 합성되어 있는 함수들에 대해서도 동기적으로 동작할 수 있게 된다.

하지만 위의 방법 역시 첫 번째 인수가 `Promise` 객체인 경우에는 대처할 수 없다.

```javascript
go(
  Promise.resolve(1),
  a => a + 10,
  a => Promise.resolve(a + 100),
  a => a + 1000,
  log
); // [object Promise]101001000
```

이와 같은 상황에 대처하기 위해 첫 번째 인수로 `Promise` 객체가 전달되더라도 합성되어 있는 함수들을 평가하기 전에 `Promise`라면 실제 값으로 해소한 다음 전달할 필요가 생긴다.

```javascript
const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  return go1(acc, function recur(acc) {
    for (const a of iter) {
      acc = f(acc, a);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});

go(
  Promise.resolve(1),
  a => a + 10,
  a => Promise.resolve(a + 100),
  a => a + 1000,
  log
); // 1111
```

거기에 추가적으로 합성되어 있는 함수 중 평가 결과가 `Promise.reject()`인 경우에 대해 대응할 필요도 생긴다. 이때는 `catch` 문을 작성하여 예외 처리를 해주게 된다.

```javascript
go(
  Promise.resolve(1),
  a => a + 10,
  a => Promise.reject('error')
  a => a + 1000,
  log
); // Uncaught (in promise) error

go(
  Promise.resolve(1),
  a => a + 10,
  a => Promise.reject('error'),
  a => a + 1000,
  log
).catch(e => console.log(e)); // error
```

## 지연 평가와 Promise

앞서 구현한 `L.map`,`map`,`take`는 기본적으로 동기적인 상황을 가정하고 구현을 진행하였다. 그렇기 때문에 비동기적인 상황에서도 정상적으로 동작할 수 있게끔 수정해주는 작업이 필요하다.

```javascript
go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map(a => a + 10),
  take(2),
  log
); // [object Promise]10, [object Promise]10

const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));
// 기존 L.map
L.map = curry(function* (f, iter) {
  for (const a of iter) {
    yield f(a);
  }
});

// 변경 후 L.map
L.map = curry(function* (f, iter) {
  for (const a of iter) {
    yield go1(a, f);
  }
});

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map(a => a + 10),
  take(2),
  log
);
// 0: Promise {<fulfilled>: 11}
// 1: Promise {<fulfilled>: 12}

// 기존 take
const take = curry((l, iter) => {
  const res = [];
  iter = iter[Symbol.iterator]();
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    res.push(a);
    if (res.length === l) return res;
  }
  return res;
});

// 변경 후 take
const take = curry((l, iter) => {
  const res = [];
  iter = iter[Symbol.iterator]();
  return (function recur() {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      if (a instanceof Promise)
        return a.then(a => ((res.push(a), res).length === l ? res : recur()));
      res.push(a);
      if (res.length === l) return res;
    }
  })();
});

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map(a => a + 10),
  take(2),
  log
); // [11, 12]
```

## Kleisli Composition - L.filter, filter, nop, take

`filter`에서 지연 평가와 비동기를 함께 지원하기 위해서는 Kleisli Composition을 적용해야 한다.

```javascript
go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => Promise.resolve(a * a)),
  L.filter(a => a % 2),
  take(2),
  log
); // []
```

위의 코드에서 `L.map`이 평가되어 `filter`로 전달되는 값은 `Promise` 객체이기 때문에 `Promise{} % 2` 연산이 되어 정상적으로 동작할 수 없게 된다.

```javascript
// 기존 L.filter
L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    if (f(a)) yield a;
  }
});

// 변경 후 L.filter
L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    const b = go1(a, f);
    if (b) yield a;
  }
});

go(
  [1, 2, 3, 4, 5, 6],
  L.filter(a => a % 2),
  take(2),
  log
); //[1, 3]
go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => Promise.resolve(a * a)),
  L.filter(a => a % 2),
  take(2),
  log
); //[1, 4]
```

하지만 `L.filter`만 단독으로 사용할 경우(전달받은 값이 `Promise`가 아닌 경우)에는 정상적으로 동작하지만, `Promise`를 전달받게 되면 `L.filter` 내부의 `b`가 `Promise` 객체인 것을 확인할 수 있고 `Promise` 객체는 `truthy`로 평가되어 문제가 발생하는 것이다.

```javascript
const nop = Symbol('nop');
L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    const b = go1(a, f);
    if (b instanceof Promise) yield b.then(b => (b ? a : Promise.reject(nop)));
    else if (b) yield a;
  }
});
```

`b`가 `Promise`라면 `then` 메서드를 이용해 `b`의 값을 해소하게 되는데 이때 해소된 값이 `true`인 경우에는 `a`를 `resolve`하게 된다. 이때 만약 `a`가 `Promise`라 할지라도 `then` 메서드 내부에서 반환될 때는 `resolve`된 상태로 전달되기 때문에 가능한 것이다.

중요한 것은 `b`가 `resolve`된 값이 `false`인 경우 아무 행동도 하지 않도록 만들어야 하는데, `yield`는 무조건 발생하는 상황이다. 따라서 `reject`를 발생시키는데 이때 `reject`만 진행하면 아무 행동도 수행하지 않는 용도인지 에러 발생을 위한 용도인지 구분할 수 없기 때문에 `Symbol`을 사용하게 된다.

그 다음 추가적으로 `take`에 대해서도 `reject`에 대한 처리를 해주어야 한다.

```javascript
const take = curry((l, iter) => {
  const res = [];
  iter = iter[Symbol.iterator]();
  return (function recur() {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      if (a instanceof Promise)
        return a
          .then(a => ((res.push(a), res).length === l ? res : recur()))
          .catch(e => (e === nop ? recur() : Promise.reject(e)));
      res.push(a);
      if (res.length === l) return res;
    }
    return res;
  })();
});
```

`take`에서 `reject`가 `catch`되더라도 해당 `reject`가 `nop`인 경우에는 무시하고 평가를 이어가게 된다.

## reduce에서 nop 지원

```javascript
go(
  [1, 2, 3, 4, 5],
  L.map(a => Promise.resolve(a * a)),
  L.filter(a => Promise.resolve(a % 2)),
  reduce(add),
  log
); // 1[object Promise][object Promise][object Promise] Uncaught (in promise) Symbol(nop)
```

```javascript
// 기존 reduce
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  return go1(acc, function recur(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      acc = f(acc, a);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});

// 변경 후 reduce
const reduceF = (acc, a, f) =>
  a instanceof Promise
    ? a.then(
        a => f(acc, a),
        e => (e === nop ? acc : Promise.reject(e))
      )
    : f(acc, a);

const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter.next().value;
  }
  return go1(acc, function recur(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      acc = reduceF(acc, cur.value, f);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});
```

결국은 `reduceF` 함수에서 `a`가 `Promise`인 경우에는 `then`을 통해 `a`를 `resolve`한 다음 `f(acc, a)`를 진행하게 되고, `reject(nop)`인 경우에는 `acc`를 그대로 반환하게 된다. 물론 `Promise`가 아닌 경우에는 기존과 동일하게 `f(acc, a)`를 이어가게 된다.

## 지연된 함수열을 병렬적으로 평가하기 - C.reduce, C.take

자바스크립트가 기본적으로 싱글 스레드 환경이기 때문에 병렬 프로그래밍을 할 일이 없다고 생각하기 쉽지만, 자바스크립트에서 어떤 로직을 제어하는 것을 싱글 스레드로 비동기적으로 제어하는 것일 뿐이기 때문에 병렬적인 프로그래밍 구성도 충분히 가능하다.

Node.js의 경우 네트워크나 기타 I/O로 작업을 요청하고 대기하는 시점을 관리해주는 것이기 때문에 요청들을 동시에 보내서 하나의 로직으로 귀결시키는 것은 충분히 고려할만한 사항이다.

```javascript
const delay500 = a => new Promise(resolve => setTimeout(() => resolve(a), 500));
go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => delay500(a * a)),
  L.filter(a => a % 2),
  reduce(add),
  log
);
```

만약 `L.map`,`L.filter` 같은 지연 평가가 아닌 즉시 평가(`map` 등)이라면 평가의 순서는 가로 방향부터 이루어져게 된다. 따라서 즉시 평가 방식의 함수로는 병렬적인 프로그래밍을 수행하기 어렵다. 다만 지연 평가를 이용하는 경우에는 평가 순서가 가로 방향이 아닌 세로 방향으로 이루어지기 때문에 각각의 값에 대해 독립적인 수행이 가능하게 된다.

위의 코드에서는 `reduce`의 경우 하나씩 값이 생성되는 것을 기다린 다음 최종적으로 누산을 진행하고 있다. 그렇다면 값에 대한 요청을 한 번에 보낸 다음 `reduce`를 수행한다면 부하는 조금 더 생길 지라도 빠르게 결과를 얻을 수 있을 것이다.

```javascript
C.reduce = curry((f, acc, iter) =>
  iter ? reduce(f, acc, [...iter]) : reduce(f, [...acc])
);
```

코드를 살펴보면 `reduce`로 대기하고 있던 함수를 스프레드 연산자를 이용하여 모두 다 실행을 한 다음 개별적으로 비동기 제어를 한 뒤 앞에서부터 누적하게 되는 것이다.

```javascript
const delay1000 = a =>
  new Promise(resolve => setTimeout(() => resolve(a), 1000));

go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => delay1000(a * a)),
  L.filter(a => delay1000(a % 2)),
  L.map(a => delay1000(a * a)),
  C.reduce(add),
  log
);
```

하지만 위 코드를 실행하면 `catch`되지 않은 부분이 있다고 출력되지만 결과는 정상적으로 나온다. `reject`가 발생하면 우선적으로 `catch`되지 않은 부분이 있다고 출력하는 현상때문이다. 따라서 우선은 `catch`의 효과만 부여하고 추후에 비동기적으로 해당 예외를 처리해줄 것이라는 것을 명시해준다.

!> 이 부분도 Promise 공부 끝나면 다시 정리하기.

```javascript
C.reduce = curry((f, acc, iter) => {
  const iter2 = iter ? [...iter] : [...acc];
  iter2.forEach(a => a.catch(function () {}));
  return iter ? reduce(f, acc, iter2) : reduce(f, iter2);
});

function noop() {}
const catchNoop = arr =>
  arr.forEach(a => (a instanceof Promise ? a.catch(noop) : a), arr);

C.reduce = curry((f, acc, iter) => {
  const iter2 = catchNoop(iter ? [...iter], [...acc])
  return iter ? reduce(f, acc, iter2) : reduce(f, iter2)
})
```

`take` 역시 같은 방식으로 수정하면 된다.

```javascript
C.take = curry((l, iter) => take(1, catchNoop([...iter])));
```

## 즉시 병렬적으로 평가하기 - C.map, C.filter

```javascript
C.take = curry((l, iter) => take(1, catchNoop([...iter])));
C.takeAll = C.take(Infinity);

C.map = curry(pipe(L.map, C.takeAll));
C.filter = curry(pipe(L.filter, C.takeAll));

C.map(a => delay500(a * a), [1, 2, 3, 4]).then(console.log); // [1, 4, 9, 16]
C.filter(a => delay500(a % 2), [1, 2, 3, 4]).then(console.log); // [1, 3]
```
