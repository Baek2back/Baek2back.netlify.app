# 부분집합(Subset)

프로그래머스 Level 2 [후보키](https://programmers.co.kr/learn/courses/30/lessons/42890) 문제를 풀던 도중 현재 배열 내에서 특정 요소를 부분집합으로 갖는 요소를 모두 제거해야 하는 함수를 구현하였다.

```javascript
const src = [0, 2]; // ← 해당 요소를 부분집합으로 갖는 요소 찾기.

const targets = [
  [0, 1],
  [0, 1, 2],
  [0, 2],
  [0, 3],
  [1, 2],
  [1, 3]
  // ...
];
```

처음에는 모두 `join(' ')`을 이용하여 `string`으로 만들고 `targets`에 있는 요소(ex. `target: 0 2`) 중에 `target.includes(src)`를 `true`로 반환하는 케이스를 이용하려 하였으나 `0 1 2`의 경우 `src`를 포함하고 있다고 인지하지 못하는 경우가 발생하였다.

```javascript
const src = '0 2';

const targets = [
  '0 1',
  '0 1 2', // ← target.include(src) === false
  '0 2', // ← OK
  '0 3',
  '1 2',
  '1 3'
  // ...
];
```

따라서 `src`의 모든 요소가 `target`에 포함되어 있다면 `src`가 `target`의 부분집합(subset)인 점을 이용하여 `Array.prototype.every()`를 적용한 함수로 구현하였다.

```javascript
const isSubset = (src, target) => {
  return src.every((element) => target.includes(element));
};
const result = targets.filter((target) => !isSubset(src, target));

console.log(result);
/*
  [0, 1],
  [0, 3],
  [1, 2],
  [1, 3]
  ...
 */
```
