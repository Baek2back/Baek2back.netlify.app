# 순열과 조합(Permutations and Combinations)

## 조합(Combinations)

<sub>4</sub>C<sub>3</sub>을 코드로 구현해보자.

```javascript
Input: [1, 2, 3, 4];
Output: [
  [1, 2, 3],
  [1, 2, 4],
  [1, 3, 4],
  [2, 3, 4]
];
```

4 개 중에서 3개를 선택할 때 나올 수 있는 모든 조합을 구하는 것이 목적이며, 순서가 바뀌어도 같은 조합으로 취급한다.

```
start:

  ① 1을 선택함 → 나머지인 [2,3,4] 중에서 2개를 선택하는 조합을 나열한다.
  [(1),2,3], [(1),2,4], [(1),3,4]
  ② 2를 선택함 → 나머지인 [3,4] 중에서 2개를 선택하는 조합을 나열한다.
  [(2),3,4]
  ③ 3을 선택함 → 나머지인 [4] 중에서 2개를 선택하는 조합을 나열한다.
  []
  ④ 4를 선택함 → 나머지인 [] 중에서 2개를 선택하는 조합을 나열한다.
  []

end:
```

```javascript
const getCombinations = (arr, toSelectNumber) => {
  if (toSelectNumber === 1) return arr.map((value) => [value]);
  // 1개를 선택하는 경우에는 배열 내에 있는 요소를 각각 선택하는 것과 동일하다.
  // ex) [1,2,3] → [[1],[2],[3]]

  return arr.reduce((result, fixed, index, origin) => {
    const rest = origin.slice(index + 1); // 선택된 fixed를 제외한 나머지
    const combinations = getCombinations(rest, toSelectNumber - 1); // 나머지에서 현재 뽑아야 하는 개수의 보다 한 개 더 적게 선택해야 한다.
    const attached = combinations.map((combination) => [fixed, ...combination]);
    // 반환된 결과에 선택해둔 fixed를 앞에 붙여주어야 한다.
    return (result = [...result, ...attached]);
  }, []);
};

const example = [1, 2, 3, 4];
const result = getCombinations(example, 3);
// [[1,2,3],[1,2,4],[1,3,4],[2,3,4]]
```

## 순열(Permutations)

조합은 **순서**에 관계없이 선택한 방식이라면, 순열에서는 **순서**가 중요하다.

> <sub>n</sub>P<sub>r</sub> = <sub>n</sub>C<sub>r</sub> x r!<br>
> 조합을 구한 후, 뽑고자 하는 수의 factorial을 곱하면 순열을 구하는 공식이다.

`[1,2,3,4]`에서 3개를 뽑아 순열을 만든다고 해보자.

```
[1,2,3] => 여기서 순서를 바꾸는 경우의 수 = 3!
[1,2,4] => 여기서 순서를 바꾸는 경우의 수 = 3!
[1,3,4] => 여기서 순서를 바꾸는 경우의 수 = 3!
[2,3,4] => 여기서 순서를 바꾸는 경우의 수 = 3!
```

```
1(fixed) → permutation([2,3,4])
         → 2(fixed) → permutation([3,4])
                    → 3(fixed) → permutation([4])
2(fixed) → permutation([1,3,4]) ...
3(fixed) → permutation([1,2,4]) ...
4(fixed) → permutation([1,2,3]) ...
```

그렇다면 이제 기존 조합 함수에서 나머지인 `rest`를 구하는 부분을 fixed 된 위치를 제외한 배열로 만들어주면 된다.

```javascript
const rest = [...origin.slice(0, index), ...origin.slice(index + 1)];
```

```javascript
const getPermutations = (arr, toSelectNumber) => {
  if (toSelectNumber === 1) return arr.map((value) => [value]);

  return arr.reduce((result, fixed, index, origin) => {
    const rest = [...origin.slice(0, index), ...origin.slice(index + 1)];
    const permutations = getPermutations(rest, toSelectNumber - 1);
    const attached = permutations.map((permutation) => [fixed, ...permutation]);
    return (result = [...result, ...attached]);
  }, []);
};

const example = [1, 2, 3, 4];
const result = getPermutations(example, 3);
/*
  [1,2,3], [1,2,4],
  [1,3,2], [1,3,4],
  [1,4,2], [1,4,3],
  [2,1,3], [2,1,4],
  [2,3,1], [2,3,4],
  [2,4,1], [2,4,3],
  [3,1,2], [3,1,4],
  [3,2,1], [3,2,4],
  [3,4,1], [3,4,2],
  [4,1,2], [4,1,3],
  [4,2,1], [4,2,3],
  [4,3,1], [4,3,2]
*/
```

## 중복조합·중복순열

중복조합과 중복순열은 기존 순열과 조합 알고리즘에서 중복을 허용하게끔만 바꿔주면 된다.

따라서 기존 코드에서 `rest`를 구하는 부분만 수정해주면 가볍게 해결된다. 앞으로 고를 숫자의 갯수는 기존과 마찬가지로 1 감소시키면서 골라야 할 대상은 `fixed`를 제외하지 않게끔 만들어주면 되기 때문이다.

```javascript
// 중복 조합
...
  return arr.reduce((result, fixed, index, origin) => {
    const rest = origin.slice(index); // 선택된 fixed를 포함시킨다.
    ...
  }, []);
...

// 중복 순열
...
  return arr.reduce((result, fixed, index, origin) => {
    const rest = [...origin.slice(0)]; // 선택된 fixed를 포함시킨다.
    ...
  }, []);
...
```

## Reference

- [JavaScript로 순열과 조합 알고리즘 구현하기](https://medium.com/@jun.choi.4928/javascript%EB%A1%9C-%EC%88%9C%EC%97%B4%EA%B3%BC-%EC%A1%B0%ED%95%A9-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-21df4b536349)
