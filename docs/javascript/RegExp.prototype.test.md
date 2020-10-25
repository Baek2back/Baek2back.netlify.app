# RegExp.prototype.test()

## 전역 플래그 설정 시 test()의 동작 방식

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

정규 표현식에 부합하는 문자열만 필터링하기 위해 두 가지의 메서드를 떠올렸는데, 이상하게도 `test()`를 적용했을 때 배열이 제대로 필터링되지 않는 것을 화인하였고, [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) 레퍼런스를 확인하던 도중, 그 해답을 얻을 수 있었다.

> **전역 플래그와 `test()`**

정규 표현식에 전역 플래그( _g_ )를 설정한 경우, `test()` 메서드는 정규 표현식의 _lastIndex_ 를 업데이트한다.(`RegExp.prototype.exec()` 역시 _lastIndex_ 속성을 업데이트한다.)

따라서 동일한 정규 표현식을 이용하여 `test(str)`을 다시 호출하게 되면 `str`에 대한 검색을 새롭게 갱신된 _lastIndex_ 부터 진행하게 되는 것이다.또한 _lastIndex_ 속성은 `test()`가 `true`를 반환할 때마다 증가하게 된다.

!> `test()`가 `false`를 반환하기 이전까지는 _lastIndex_ 는 초기화되지 않는다. 동일한 정규 표현식에 대해 다른 문자열을 매개변수로 제공하여도 _lastIndex_ 는 초기화되지 않는다. 다만 `test()`가 `false`를 반환하면 _lastIndex_ 속성은 0으로 초기화된다.

## Conclusion

초기 의도대로 동작하도록 하려면 정규 표현식에 전역 플래그를 제거한 뒤 `RegExp.prototype.test()`를 사용하거나 전역 플래그를 이용하여 처리하고 싶다면 `String.prototype.match()`를 사용하여 해결하면 된다.

## Reference

- https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/RegExp/tes
