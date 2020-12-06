# 배열 비교하기

프로그래머스 [여행경로](https://programmers.co.kr/learn/courses/30/lessons/43164) 문제를 풀던 도중 가능한 경로를 배열에 담아두고, 알파벳 순으로 경로들을 비교하여 정렬해야 하는 상황이 생겼다.

```javascript
const routes = [
  ['a', 'c', 'd', 'b'],
  ['a', 'b', 'c', 'd']
];
routes.sort(compare);
/*
[
  ['a','b','c','d'],
  ['a','c','d','b']
]
*/
```

위와 같이 비교의 대상이 배열 단위로 이루어지기 때문에 배열 간의 비교를 수행하는 `compare` 함수를 정의하였다.

```javascript
const compare = (arr1, arr2) => {
  let idx = 0;
  while (arr1[idx] === arr2[idx]) idx += 1;
  if (arr1[idx] < arr2[idx]) return -1;
  if (arr1[idx] > arr2[idx]) return 1;
  return 0;
};

const routes = [
  ['a', 'b', 'a'],
  ['a', 'b', 'a']
];

routes.sort(compare); // 시간 초과
```

하지만 위와 같이 동일한 배열 요소들을 비교할 때 시간 초과가 발생하였고, 디버깅을 통해 문제를 확인할 수 있었다.

```javascript
const arr1 = [1, 2, 3];
const arr2 = [1, 2, 3];
arr1[arr1.length] === arr2[arr2.length];
// undefined === undefined → true
```

배열 역시 객체이기 때문에 존재하지 않는 프로퍼티를 참조하면 `undefined`를 반환하게 된다. 이때 처음 의도는 두 배열의 요소를 비교했을 때, 같은 요소들은 건너뛰게 하도록 `arr1[idx] === arr2[idx]`라는 조건식을 정의하였는데, 두 배열이 모두 같은 요소로 구성되어 있는 경우 배열의 모든 요소를 순회한 이후에도 `undefined`를 반환하였기 때문에 발생한 문제였다.

따라서 `idx`가 배열의 길이와 같아지게 되면 0을 반환하게끔 수정하였으나, 꼬리 재귀 형태로 재귀 호출이 가능할 것이라 판단되어 재귀 함수로 수정하였다. 또한 `Array.prototype.sort` 메서드의 콜백 함수로 주입되는 비교 함수의 매개변수가 2개이므로, Point-Free하게 사용할 수 있게끔 `default parameter`를 이용하여 구현하였다.

```javascript
const compare = (arr1, arr2, idx = 0, limit = arr1.length) => {
  // 두 배열의 길이가 동일하다는 전제가 필요하다.
  if (idx === limit) return 0;
  if (arr1[idx] === arr2[idx]) return compare(arr1, arr2, idx + 1);
  return arr[idx] < arr2[idx] ? -1 : 1;
};
```
