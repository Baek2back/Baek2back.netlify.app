# JSON.parse()

**JSON**(**JavaScript Object Notation**)은 클라이언트와 서버 간의 HTTP 통신을 위한 텍스트 데이터 포맷이다.자바스크립트에 종속되지 않는 언어 독립형 데이터 포맷으로,대부분의 프로그래밍 언어에서 사용 가능하다.

`JSON.parse()`는 JSON 형태의 문자열을 객체로 변환한다.서버로부터 클라이언트에 전송된 JSON 데이터는 문자열이다. 따라서 이 문자열을 객체로 사용하기 위해서는 JSON 포맷의 문자열을 객체화해야 하는데 이를 **역직렬화**(**deserializing**)라 한다.

!> 클라이언트가 서버로 객체를 전송하기 위해서는 객체를 문자열화하는 작업을 해야 하는데 이는 **직렬화**(**serializing**)라 하고, 자바스크립트에서 객체를 JSON 포맷의 문자열로 변환하는 것은 `JSON.stringify` 메서드를 이용하면 된다.

## Syntax

> JSON.parse(text[, _reviver_])

## Examples

```javascript
JSON.parse('{}'); // {}
JSON.parse('true'); // true
JSON.parse('"foo"'); // "foo"
JSON.parse('[1,5,"false"]'); // [1,5,"false"]
JSON.parse('null'); // null
```

> **_reviver_**

앞서 Syntax에서 _reviver_ 는 _Optional_ 이다. 만약 _reviver_ 가 제공되면 파싱된 값은 return되기 전에 변형(transform)된다. 만약 _reviver_ 가 `undefined`를 반환(또는 함수가 종료되기 이전에 실행이 중단되는 등의 이유로 어떠한 값도 반환하지 않는 경우)하면 해당 프로퍼티는 반환되는 `object`에서 제거된다.

```javascript
JSON.parse('{"p":5}', (key, value) => {
  return typeof value === 'number' ? value * 2 : value;
});

// { p: 10 }

JSON.parse('{"1": 1, "2": 2, "3": {"4": 4, "5": {"6": 6}}}', (key, value) => {
  if (key === '2') return;
  return value;
});

// {1: 1, 3: {4: 4, 5: {6: 6}}}
```

순회 방식도 무조건 순차적으로 진행되는 것이 아니라 해당 프로퍼티가 중첩된 값을 갖고 있다면 중첩된 값을 우선적으로 순회하게 된다.

```javascript
JSON.parse('{"1": 1, "2": 2, "3": {"4": 4, "5": {"6": 6}}}', (key, value) => {
  console.log(key); // log the current property name, the last is "".
  return value; // return the unchanged property value.
});

// 1 → 2 → 4 → 6 → 5 → 3 → ""
```

!> `JSON.parse()`는 _trailing commas_ 와 _single quotes_ 를 허용하지 않는다.

```javascript
JSON.parse('[1,2,3,4,]'); // SyntaxError (trailing commas)
JSON.parse('{"foo": 1, }'); // SyntaxError(trailing commas)
JSON.parse("{'foo': 1}"); // SyntaxError(single quotes)
```

## Conclusion

[튜플](https://programmers.co.kr/learn/courses/30/lessons/64065) 문제를 풀다가 `'{{123}}'` 형태의 문자열을 `'[[123]]'` 형태의 배열 객체로 변환하면 취급하기 편할 것이라 생각하였다. 그러기 위해서는 우선 문자열을 배열의 형태로 변환하고 변환된 문자열을 자바스크립트 객체로 변환하는 방법이 필요했는데 그때 찾은 메서드가 `JSON.parse()`였다. _reviver_ 에 함수를 제공하면 각각의 _key, value_ 를 활용하여 다양하게 응용하는 방법이 가능할 것 같다.

## Reference

- [[MDN]:JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)
- [모던 자바스크립트 Deep Dive 43.2절 - JSON](http://www.yes24.com/Product/Goods/92742567)
