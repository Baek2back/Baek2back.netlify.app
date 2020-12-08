# 함수형 자바스크립트

> ES5에 도입된 배열 고차 함수

1. `Array.prototype.map`
2. `Array.prototype.filter`
3. `Array.prototype.reduce`
4. `...etc`

위의 고차 함수들은 기본적으로 `Array.prototype`에 정의되어 있기 때문에 `Array-Like` 객체나 `object({})`에는 적용할 수 없다.

> `Array-Like` 객체

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
  for (const a of iter) {
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
  for (const a of iter) f(a) && res.push(a);
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
  for (const a of iter) acc = f(acc, a);
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
  for (const a of iter) acc = f(acc, a);
  return acc;
};

reduce((total_price, product) => total_price + product.price, 0, products); // 105000
reduce((total_price, product) => total_price + product.price, products); // 105000
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
  console.log
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
  console.log
); // 112

// pipe의 경우는 다음과 같이 사용해야 하는 제약이 있다.
log(f(add(0, 1)));

// 다음과 같이 사용할 수 있도록 수정해보자.
const f = pipe(
  (a, b) => a + b,
  a => a + 1,
  a => a + 10,
  a => a + 100
);
console.log(f(0, 1)); // 112

const pipe = (f, ...fs) => (...args) => {
  return go(f(...args), ...fs);
};
```

## curry

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
  for (const a of iter) {
    res.push(f(a));
  }
  return res;
});

const filter = curry((f, iter) => {
  const res = [];
  for (const a of iter) f(a) && res.push(a);
  return res;
});

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
log(list); // [0,1,2,3,4]
log(reduce(add, list)); // 6

const L = {};
L.range = function* (length) {
  let i = -1;
  while (++i < length) yield i;
};

let list = L.range(5);
log(list);
// GeneratorFunctionPrototype {...}
log(reduce(add, list)); // 6

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
  for (const a of iter) {
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
  ].flatMap((a) => a.map((a) => a + a))
);
// [2,4,6,8,10,12,14]
log(
  flatten(
    [
      [1, 2],
      [3, 4],
      [5, 6, 7]
    ].map((a) => a.map((a) => a + a))
  )
);
// [2,4,6,8,10,12,14]

L.flatMap = curry(pipe(L.map, L.flatten));
```