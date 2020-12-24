# 배열 K번 회전하기

주어진 배열을 `k`번 왼쪽으로 회전하는 함수를 만들어보자.

예를 들어 `[1,2,3,4,5,6,7]`을 2번 회전하면 `[3,4,5,6,7,1,2]`가 될 것이다.

## 접근 1: 직접 회전시키기

```javascript
const rotateToLeft = (array, times) => {
  if (!times) return array;
  const [first, ...rest] = array;
  return rotateToLeft([...rest, first], times - 1);
};

const arr = [1, 2, 3, 4, 5, 6, 7];
rotateToLeft(arr, 2); // [3, 4, 5, 6, 7, 1, 2]
```

`rotateToLeft` 함수의 인수로 전달되는 `times` 즉, `k`가 작은 수라면 문제없이 동작하겠지만, `k`가 큰 수였다면 `RangeError`가 발생한다는 한계가 존재한다.

그렇다면 배열을 `k`번만큼 회전시키지 않고 `array`와 `times`만을 이용해서 구할 수 있는 방법이 있지 않을까?

## 접근 2: O(N)

> `k = 3` 인 경우

- original : `[1, 2, 3, 4, 5, 6, 7]`
- 배열을 `k`개의 수와 나머지로 분리: `[[1, 2, 3], [4, 5, 6, 7]]`
- 배열을 뒤집고 펼치기: `[[4, 5, 6, 7], [1, 2, 3]] → [4, 5, 6, 7, 1, 2, 3]`

> `k`가 배열의 길이보다 크거나 같은 경우

만약, `k`를 8이라 하면 7개의 요소를 가진 배열에서 8개를 선택하는 것은 불가능하므로 `k = k % array.length`를 먼저 처리한 뒤 `k`개를 선택해야 한다.

```javascript
const rotateToRight = (array, times) => {
  times = times % array.length;
  if (!times) return array;
  const origin = [...array];
  const picked = origin.slice(0, times);
  const remainder = origin.slice(times);
  return [...remainder, ...picked];
};

const arr = [1, 2, 3, 4, 5, 6, 7];
rotateToRight(arr, 3); // [4, 5, 6, 7, 1, 2, 3]
```

## 배열을 오른쪽으로 회전시키려면 어떻게 해야 할까?

> `k = 3`인 경우

- original: `[1, 2, 3, 4, 5, 6, 7]`
- 배열을 `n - k`개의 수(4)와 나머지로 분리: `[[1, 2, 3, 4], [5, 6, 7]]`
- 배열을 뒤집고 펼치기: `[[5, 6, 7], [1, 2, 3, 4]] → [5, 6, 7, 1, 2, 3, 4]`

```javascript
const rotateToRight = (array, times) => {
  times = times % array.length;
  if (!times) return array;
  const range = array.length - times;
  const origin = [...array];
  const picked = origin.slice(0, range);
  const remainder = origin.slice(range);
  return [...remainder, ...picked];
};

const arr = [1, 2, 3, 4, 5, 6, 7];
rotateToRight(arr, 3); // [5, 6, 7, 1, 2, 3, 4]
```

## 최종 구현

```javascript
const rotate = (array, times, direction = 'left') => {
  times = times % array.length;
  if (!times) return array;
  const origin = [...array];
  const pickedRange = direction === 'left' ? times : array.length - times;
  const picked = origin.slice(0, pickedRange);
  const remainder = origin.slice(pickedRange);
  return [...remainder, ...picked];
};
```
