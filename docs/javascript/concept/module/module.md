## 모듈이란?

모듈(module)이란 애플리케이션을 구성하는 개별적 요소로서 재사용 가능한 코드 조각을 의미한다. 일반적으로는 기능을 기준으로 파일 단위로 분리한다. 이때 모듈이 되기 위해서는 자신만의 **파일 스코프**(모듈 스코프 )를 가질 수 있어야 한다.

자신만의 파일 스코프를 갖는 모듈의 자산(변수,함수,객체 등)은 기본적으로 비공개 상태로 캡슐화되기 때문에 다른 모듈에서는 접근할 수 없다. 즉, 모듈은 개별적 존재로서 애플리케이션과 분리되어 존재한다.

?> 모듈은 공개가 필요한 자산에 한정하여 명시적으로 선택적인 공개가 가능하다.

## 자바스크립트와 모듈

자바스크립트 파일을 여러 개의 파일로 분리하여 `script` 태그를 이용하여 로드한다 하더라도 분리된 자바스크립트 파일들은 결국 하나의 전역을 공유하게 되므로, 분리된 자바스크립트 파일들의 전역 변수가 중복되는 등의 문제가 발생한다.

```html
<html>
  <body>
    <script src="./app.js"></script>
    <script src="./main.js"></script>
  </body>
</html>
```

```javascript
// app.js
var num = 10;
function getNum() {
  console.log(num);
}

// main.js
var num = 20;
function getNum() {
  console.log(num);
}
```

```html
<script>
  getNum(); // 20
</script>
```

두 파일이 모두 전역 공간을 공유하기 때문에 나중에 로드된 `main.js`의 코드가 우선시된다. 이러한 문제는 복잡한 애플리케이션을 개발할 때 변수를 중복 선언하거나 의도치 않은 값의 변경을 유발할 수 있다.

자바스크립트를 브라우저 환경에 국한하지 않고 범용적으로 사용하기 위해 모듈 시스템은 반드시 해결해야 하는 문제였고, AMD나 CommonJS의 등장으로 이어졌다.

자바스크립트 런타임 환경인 Node.js는 CommonJS를 채택하였고, 사양과 100% 동일하지는 않지만 기본적으로는 CommonJS 사양을 따르고 있다. 따라서 파일 별로 독립적인 파일 스코프(모듈 스코프)를 갖는다.

## ES6 모듈(ESM)

ES6부터는 클라이언트 사이드 자바스크립트에서도 동작하는 모듈 기능이 추가되었다. 현재 IE를 제외한 대부분의 브라우저에서 ES6를 사용할 수 있다.

- `export` 키워드를 사용하면 외부 모듈에서 해당 변수나 함수에 접근 가능하다.
- `import` 키워드를 사용하면 외부 모듈의 기능을 가져올 수 있다.

```javascript
// 📁 sayHi.js
export function sayHi(user) {
  alert(`Hello, ${user}!`);
}
```

```javascript
// 📁 main.js
import { sayHi } from './sayHi.js';

sayHi('John'); // Hello, John!
```

브라우저에서 모듈을 사용하려면 script 태그에 `type="module"` 어트리뷰트를 추가하면 해당 스크립트가 모듈이라는 것을 브라우저가 인식할 수 있게 된다.

```html
<!-- index.html -->
<script type="module">
  import { sayHi } from './say.js';
  document.body.innerHTML = sayHi('John');
</script>
```

```javascript
// say.js
export function sayHi(user) {
  return `Hello, ${user}!`;
}
```

!> 모듈은 로컬 파일 시스템(`file://` 프로토콜)에서는 동작하지 않는다. 오직 HTTP(s) 프로토콜에서만 동작한다.

## 핵심 기능

### use strict

모듈은 항상 strict 모드로 실행되며, 선언되지 않은 변수에 값을 할당하는 등의 코드는 에러를 발생시킨다. 또한 모듈 최상위 레벨의 `this`는 `undefined`다.

```html
<script type="module">
  alert(this); // undefined
  a = 5; // Error
</script>
```

### 단 한 번만 평가됨

동일한 모듈이 여러 곳에서 사용되더라도 모듈은 최초 호출 시 단 한 번만 실행된다. 실행 이후의 결과는 해당 모듈을 `import`하는 모든 모듈에게 공유된다.

```javascript
// 📁 alert.js
alert('모듈이 평가되었습니다.');
```

```javascript
// 동일한 모듈을 여러 모듈에서 가져오기

// 📁 1.js
import './alert.js'; // alert("모듈이 평가되었습니다.")

// 📁 2.js
import './alert.js'; // 아무 일도 발생하지 않음.
```

실제로는 최상위 레벨 모듈을 일반적으로 초기화나 내부 데이터 구조를 만들 때 사용하며, 이것들을 내보내 재사용하게 된다.

```javascript
// 📁 admin.js
export let admin = {
  name: 'John'
};
```

이 모듈을 가져오는 모듈이 여러 개라 하더라도 모듈은 **최초 호출 시 단 한 번만 평가되므로**, `admin` 객체가 만들어진 이후에는 해당 모듈을 가져오는 모든 모듈에 `admin` 객체가 전달된다. 즉, 각 모듈에 동일한 `admin` 객체가 전달되는 것이다.

```javascript
// 📁 1.js
import { admin } from './admin.js';
admin.name = 'Pete';

// 📁 2.js
import { admin } from './admin.js';
alert(admin.name); // Pete

/**
 * 1.js와 2.js 모두 같은 객체를 가져오므로
 * 1.js에서 객체에 조작을 하더라도 2.js에서 확인이 가능하다.
 */
```

이러한 특징을 활용하면 모듈 설정을 쉽게 할 수 있다. 최초로 실행되는 모듈의 객체 프로퍼티를 원하는 대로 설정해주면 다른 모듈에서 해당 설정을 공유받을 수 있기 때문이다.

다음 `admin.js` 모듈은 어떠한 특정한 기능을 제공해주는데, 이 기능을 사용하려면 외부에서 `admin` 객체와 관련된 인증 정보가 필요하다고 가정해보자.

```javascript
// 📁 admin.js
export let admin = {};

export function sayHi() {
  alert(`${admin.name}님, 안녕하세요!`);
}
```

이때 최초로 실행되는 스크립트인 `init.js`에서 `admin.name`을 설정해주었다. 그러면 `admin.js`를 포함한 외부 스크립트에서 `admin.name`을 참조할 수 있게 된다.

```javascript
// 📁 init.js
import { admin } from './admin.js';
admin.name = 'Pete';

// 📁 other.js
import { admin, sayHi } from './admin.js';
alert(admin.name); // Pete
sayHi(); // Pete님, 안녕하세요!
```

### import.meta

`import.meta` 객체는 현재 모듈에 대한 정보를 제공해준다.

호스트 환경에 따라 다르지만 브라우저 환경에서는 스크립트의 URL 정보를 얻을 수 있다. 이때 HTML 내부에 있는 모듈이라면 현재 실행 중인 웹페이지의 URL 정보를 얻을 수 있다.

```html
<script type="module">
  alert(import.meta.url); // 인라인 스크립트가 위치해 있는 html 페이지 URL
</script>
```

## 브라우저에서의 기능

### 지연 실행

모듈 스크립트는 항상 지연 실행되며, `defer` 어트리뷰트를 붙인 것처럼 동작한다. 따라서 모듈 사용 시에는 DOM 파싱이 완료된 이후에 모듈이 실행된다는 점에 유의해야 한다.

### 인라인 스크립트의 비동기 처리

모듈이 아닌 일반 스크립트에서 `async` 속성은 외부 스크립트를 불러올 때만 유효하다. `async` 어트리뷰트가 정의된 스크립트는 로딩이 끝나면 DOM 파싱 완료 여부와 상관없이 바로 실행된다.

반면, 모듈 스크립트에서는 `async` 어트리뷰트를 인라인 스크립트에도 적용할 수 있다. 이러한 특징을 이용하여 광고나 document 레벨 이벤트 리스너, 카운터처럼 어디에도 종속되지 않는 기능을 구현할 때 유용하게 사용할 수 있다.

```html
<script async type="module">
  import { counter } from './analytics.js';
  counter.count();
</script>
```

### 외부 스크립트

1. `src` 어트리뷰트가 동일한 외부 스크립트는 한 번만 실행된다.

```html
<!-- my.js는 한 번만 로드되고 실행된다. -->
<script type="module" src="my.js"></script>
<script type="module" src="my.js"></script>
```

2. 외부 사이트와 같은 다른 오리진에서 모듈 스크립트를 불러오기 위해서는 CORS 헤더가 필요하다. 모듈이 저장되어 있는 원격 서버가 `Access-Control-Allow-Origin: *` 헤더를 제공해야만 외부 모듈을 불러올 수 있다.

### 경로가 없는 모듈은 금지

브라우저 환경에서 `import`는 반드시 상대 혹은 절대 URL 앞에 위치해야 한다. 경로가 없는 모듈은 허용하지 않는다. Node.js나 번들링 툴은 경로 없이도 해당 모듈을 참조할 수 있는 방법을 알지만, 브라우저의 경우는 경로 없는 모듈을 지원하지 않는다.

```javascript
import { sayHi } from 'sayHi'; // Error!
// './sayHi.js'처럼 경로 정보를 지정해주어야 한다.
```

### 호환을 위한 nomodule

구형 브라우저는 `type="module"`을 해석하지 못하기 때문에 모듈 타입의 스크립트를 만나면 이를 무시하고 넘어가게 된다. `nomodule` 속성을 사용하면 이런 상황을 대비할 수 있다.

```html
<script type="module">
  alert('Morden browser');
</script>

<script nomodule>
  alert('Old browser');
</script>
```

## 모듈 사용법

### 선언부 앞에 export 붙이기

변수나 함수, 클래스를 선언할 때 맨 앞에 `export`를 붙이면 내보내기가 가능하다.

```javascript
export let sequence = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

export const MODULES_BECAME_STANDARD_YEAR = 2015;

export class User {
  constructor(name) {
    this.name = name;
  }
}
```

### 선언부와 떨어진 곳에서 export 붙이기

```javascript
// 📁 say.js

function sayHi(user) {
  alert(`Hello, ${user}!`);
}

function sayBye(user) {
  alert(`Bye, ${user}!`);
}

export { sayHi, sayBye };
```

### import \*

```javascript
// 📁 main.js
import { sayHi, sayBye } from './say.js';

sayHi('John'); // Hello, John!
sayBye('John'); // Bye, John!
```

만약 import 할 대상이 많은 경우 `import * as <obj>`처럼 객체 형태로 원하는 요소들을 가져올 수 있다.

```javascript
// 📁 main.js
import * as say from './say.js';

say.sayHi('John');
say.sayBye('John');
```

하지만 어떤 요소를 import 할 때는 대상을 구체적으로 명시하는 것이 좋다.

1. 웹팩과 같은 모던 빌드 툴은 로딩 속도를 높이기 위해 모듈들을 한데 모으는 번들링과 최적화를 수행한다. 이 과정에서 사용하지 않는 리소스가 삭제되기도 한다. 빌드 툴은 실제 사용되는 함수가 무엇인지 파악하여, 그렇지 않은 함수는 최종 번들링 결과물에 포함하지 않는다. 이 과정에서 불필요한 코드가 제거되기 때문에 빌드 결과물의 크기가 작아지게 되는데, 이러한 최적화 과정을 **Tree-Shaking**이라고 한다.
2. `say.sayHi()`보다는 `sayHi()`가 더 간결하다.
3. 어디서 어떤 것이 쓰이는지 명확하기 때문에 리팩토링이나 유지보수에 도움이 된다.

### export 'as'

`export`에서도 `as`를 사용할 수 있다.

```javascript
// 📁 say.js
export { sayHi as hi, sayBye as bye };

// 📁 main.js
import * as say from './say.js';

say.hi('John'); // Hello, John!
say.bye('John'); // Bye. John!
```

### export default

모듈은 크게 두 종류로 나뉜다.

1. 여러 개의 함수가 있는 라이브러리 형태의 모듈(ex. `say.js`)
2. 개체 하나만 선언되어 있는 모듈(`user.js`, `class User` 하나만 내보내기 함)

?> 일반적으로는 두 번째 방식을 선호하기 때문에 함수, 클래스, 변수 등의 개체는 전용 모듈 안에 구현된다. 다만, 이러한 형태의 모듈로 구현하다보면 자연스럽게 파일 개수가 많아지지만 모듈 명과 폴더를 잘 구성하면 코드 탐색이 어렵지 않으므로 크게 문제가 되지는 않는다.

모듈은 `export default`라는 특별한 문법을 지원하는데 이를 사용하면 해당 모듈에는 개체가 하나만 존재한다는 사실을 명확히 나타낼 수 있다.

```javascript
// 📁 user.js
export default class User {
  constructor(name) {
    this.name = name;
  }
}
```

이렇게 `default`와 함께 모듈을 내보내면 중괄호 없이 모듈을 가져올 수 있게 된다.

```javascript
// 📁 main.js
import User from './user.js'; // { User }가 아닌 User로 import 가능

new User('John');
```

이때 파일 당 최대 한개의 default export가 존재할 수 있으므로 내보낼 개체에 이름이 존재하지 않아도 상관없다.

```javascript
export default class {}

export default function (user) {}

export default [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
```

`export default`는 파일 당 한개만 존재하므로 해당 개체를 가져오려 하는 모듈에선 중괄호 없이도 어떤 개체를 가지고 올지 정확히 알 수 있으므로 이름이 없어도 관계 없는 것이다.

### default export의 이름에 관한 규칙

default export는 가져오기 할 때 개발자 마음대로 이름을 지정해 줄 수 있다.

```javascript
import User from './user.js';
import MyUser from './user.js';
```

## 동적으로 모듈 가져오기

모듈 경로에는 원시값 문자열만 들어갈 수 있기 때문에 함수 호출의 결과를 경로로 쓰는 것은 불가능하다.

```javascript
import ... from getModuleName(); // Error
```

또한 런타임이나 조건 부로 모듈을 불러오는 것이 불가능하다는 한계가 있다.

```javascript
if (true) {
  import ...; // 모듈을 조건부로 불러올 수 없음(Error)
}
{
  import ...; // import 문은 블록 문 내에 위치할 수 없음(Error)
}
```

이러한 제약이 생긴 이유는 `import/export`가 코드 구조의 중심을 잡아주어야 코드 구조를 분속하여 모듈을 한 곳에 모아 번들링하고, 사용하지 않는 모듈은 제거(tree-shaking)할 수 있기 때문이다.

그럼에도 불구하고 모듈을 동적으로 불러와야 할 때는 어떻게 해야 할까?

### import() 표현식

`import(module)` 표현식은 모듈을 읽고 이 모듈이 내보내는 것들을 모두 포함하는 객체를 담은 이행된 프라미스를 반환한다. 호출은 어디서나 가능하다.

```javascript
let modulePath = prompt('Which module do you want?');

import(modulePath)
  .then(obj => <모듈 객체>)
  .catch(err => <로딩 에러>);
```

`async` 함수 안에서 사용하는 것도 가능하다.

```javascript
// 📁 say.js
export function hi() {
  alert('Hi!');
}
export function bye() {
  alert('Bye!');
}
```

```javascript
let { hi, bye } = await import('./say.js');

hi();
bye();
```

만약 `say.js`에 default export가 있는 경우에는 모듈 객체의 `default` 프로퍼티를 사용하면 된다.

```javascript
// 📁 say.js
export default function () {
  alert('default!');
}
```

```javascript
let { default: say } = await import('./say.js');
```

!> 동적 import는 일반 스크립트에서도 작동하며, `import()`는 함수 호출과 문법이 유사해 보이지만 함수 호출이 아닌, `super()`와 같이 괄호를 쓰는 특별한 문법 중 하나일 뿐이다.
