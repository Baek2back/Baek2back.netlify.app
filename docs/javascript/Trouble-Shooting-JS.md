# Javascript

## 정규 표현식 전역 플래그 설정 시 match와 test의 동작 차이.

```javascript
const regExp = /^[a-zA-Z]*$/g;
const strs = ['ab', 'cd', 'e+'];

const result = strs.filter((v) => v.match(regExp));
// ['ab','cd']
const result = strs.filter((v) => regExp.test(v));
// [?, ?]
```

배열의 요소가 알파벳으로만 구성되어 있는지 확인하기 위한 작업을 진행하던 도중 확인한 현상이다.

```javascript
① String.prototype.match();
② RegExp.prototype.test();
```

[MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) 레퍼런스를 확인하던 도중, 정규 표현식에 전역 플래그(g)를 설정한 경우 test() 메서드는 정규 표현식의 lastIndex를 업데이트한다는 것을 확인하였다. 즉, 동일한 정규 표현식에 대해 계속 test()를 호출하면 lastIndex 속성이 증가하게 되어 정상적으로 동작하지 않았던 것이다. 따라서 의도대로 동작하도록 하려면 정규 표현식에 전역 플래그를 제거하거나 match()를 이용한 방식으로 해결해야 한다.
