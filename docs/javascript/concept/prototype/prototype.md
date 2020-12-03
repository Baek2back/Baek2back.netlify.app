# 프로토타입

> [!ATTENTION]
> 자바스크립트는 **프로토타입 기반**의 객체지향 프로그래밍 언어이다.

자바스크립트는 **객체 기반의 프로그래밍 언어로 자바스크립트를 구성하고 있는 거의 "모든 것"이 객체다.** 즉, 원시 타입(primitive type)의 값을 제외한 나머지 객체 타입의 값들은 모두 객체다.

## 객체지향 프로그래밍

- 실세계의 실체(사물이나 개념)를 인식하는 철학적 사고를 프로그래밍에 접목하려는 시도에서 시작
- 실체는 특징이나 성질을 나타내는 **속성**(**attribute/property**)을 가지고 있고, 속성을 이용해 실체를 인식하거나 구별한다.
- 실체를 구성하는 다양한 속성 중에서 프로그램에 필요한 속성만 간추려 내어 표현하는 것을 **추상화**(**abstraction**)라 한다.
- **속성을 통해 여러 개의 값을 하나의 단위로 구성한 복합적인 자료구조**를 객체라 하며, 독립적인 객체의 집합으로 프로그램을 표현하고자 하는 프로그래밍 패러다임이 객체지향 프로그래밍이다.

```javascript
const circle = {
  radius: 5, // 반지름

  // 원의 지름: 2r
  getDiameter() {
    return 2 * this.radius;
  },

  // 원의 둘레: 2*pi*r
  getPerimeter() {
    return 2 * Math.PI * this.radius;
  },

  // 원의 넓이: pi * r^2
  getArea() {
    return Math.PI * this.radius ** 2;
  }
};

circle;
// { radius: 5, getDiameter: ƒ, getPerimeter: ƒ, getArea: ƒ }

circle.getDiameter(); // 10
circle.getPerimeter(); // 31.4159263589793
circle.getArea(); // 78.53981633974483
```

위와 같이 객체지향 프로그래밍은 객체의 **상태**(**state**, property)를 나타내는 데이터와 상태 데이터를 조작할 수 있는 **동작**(**behavior**, method)을 하나의 논리적 단위로 묶어 생각한다.

> [!NOTE]
> 객체는 **상태 데이터와 동작을 하나의 논리적인 단위로 묶은 복합적인 자료구조**이다.

각 객체는 자신의 고유한 기능을 수행하면서 다른 객체와 관계(relationship)를 가질 수 있으며, 다른 객체와 메시지를 주고 받거나 데이터를 처리하는 것도 가능하다. 또는 객체의 상태 데이터나 동작을 상속받아 사용하기도 한다.

## 상속과 프로토타입

자바스크립트는 프로토타입을 기반으로 상속을 구현하여 불필요한 중복을 제거한다.

```javascript
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
  this.getArea = function () {
    return Math.PI * this.radius ** 2;
  };
}

// 반지름이 1인 인스턴스 생성
const circle1 = new Circle(1);

// 반지름이 2인 인스턴스 생성
const circle2 = new Circle(2);

// Circle 생성자 함수는 인스턴스를 생성할 때마다 동일한 동작을 하는
// getArea 메서드를 중복 생성하고 모든 인스턴스가 중복 소유한다.
// 따라서 getArea 메서드는 하나만 생성하여 모든 인스턴스가 공유해서 사용하는 것이 바람직하다.

circle1.getArea === circle2.getArea; // false
```

`Circle` 생성자 함수가 생성하는 모든 인스턴스는 `radius` 프로퍼티와 `getArea` 메서드를 갖는데, `radius`의 경우 인스턴스마다 일반적으로 다르게 될 것이다. 반면 `getArea` 메서드는 모든 인스턴스가 동일하므로 하나만 생성하여 모든 인스턴스가 공유할 수 있게끔 하는 것이 바람직할 것이다.

```javascript
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
}

// Circle 생성자 함수가 생성한 모든 인스턴스가 getArea 메서드를 공유해서 사용할 수 있도록 프로토타입에 추가한다.
// 프로토타입은 Circle 생성자 함수의 prototype 프로퍼티에 바인딩되어 있다.
Circle.prototype.getArea = function () {
  return Math.PI * this.radius ** 2;
};

// 인스턴스 생성
const circle1 = new Circle(1);
const circle2 = new Circle(2);

// Circle 생성자 함수가 생성한 모든 인스턴스는 부모 객체 역할을 하는 프로토타입 객체 Circle.prototype으로부터 getArea 메서드를 상속받는다.
// 즉, Circle 생성자 함수가 생성하는 모든 인스턴스는 하나의 getArea 메서드를 공유한다.
circle1.getArea === circle2.getArea; // true
```

`Circle` 생성자 함수가 생성한 모든 인스턴스는 자신의 프로토타입, 즉 상위(부모) 객체 역할을 하는 `Circle.prototype`(프로로타입 객체)의 모든 프로퍼티와 메서드를 상속받는다.

이때 `getArea` 메서드는 단 하나만 생성되어 `Circle.prototype`의 메서드로 할당되어 있으므로 `Circle` 생성자 함수가 생성하는 모든 인스턴스는 `getArea` 메서드를 상속받아 사용할 수 있다.

이처럼 생성자 함수가 생성할 모든 인스턴스가 공통적으로 사용할 프로퍼티나 메서드를 프로토타입 객체(`.prototype`)에 미리 구현해 두면 생성자 함수에 의해 생성된 모든 인스턴스는 별도의 구현 없이도 상위(부모) 객체인 프로토타입 객체의 자산을 공유하여 사용할 수 있다.

## 프로토타입 객체

모든 객체는 `[[Prototype]]`이라는 내부 슬롯을 가지며, 이 내부 슬롯의 값은 프로토타입의 참조(`null`인 경우도 존재)다. 따라서 객체가 생성될 때 객체 생성 방식에 따라 프로토타입이 결정되고 `[[Prototype]]`에 저장된다.

> [!NOTE] **객체 리터럴에 의해 생성된 객체의 프로토타입**은 `Object.prototype`이고, **생성자 함수에 의해 생성된 인스턴스의 프로토타입**은 생성자 함수의 `prototype` 프로퍼티에 바인딩되어 있는 객체다.

모든 객체는 하나의 프로토타입을 가지며, 이때 모든 프로토타입은 생성자 함수와 연결되어 있다.

> [!ATTENTION] `[[Prototype]]` 내부 슬롯의 값이 `null`인 객체는 프로토타입이 없지만, 명시적으로 `null`이라는 값을 갖고 있기 때문에 모든 객체는 하나의 프로토타입을 가진다고 표현한 것이다.

<center><img src="docs/javascript/concept/prototype/relationship-with-object-prototype.png" alt="relationship-with-object-prototype"></center>

따라서 `[[Prototype]]` 내부 슬롯에는 직접 접근할 수는 없지만, 위에서와 같이 `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입, 즉 자신의 `[[Prototype]]` 내부 슬롯이 가리키는 프로토타입에 간접적으로 접근할 수 있다. 또한 프로토타입 객체는 자신의 `constructor` 프로퍼티를 통해 생성자 함수에 접근할 수 있고, 생성자 함수는 자신의 `prototype` 프로퍼티를 통해 프로토타입 객체에 접근할 수 있다.

### `__proto__` 접근자 프로퍼티

**모든 객체는 `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입, 즉 `[[Prototype]]` 내부 슬롯에 간접적으로 접근**할 수 있다.

<center><img src="docs/javascript/concept/prototype/proto-chrome.png" alt="proto-chrome"></center>

위에서 빨간 박스로 표시한 것은 `person` 객체의 프로토타입인 `Object.prototype`이다. 이는 `__proto__` 접근자 프로퍼티를 통해 `person` 객체의 `[[Prototype]]` 내부 슬롯이 가리키는 객체인 `Object.prototype`에 접근한 결과를 표시한 것이다. 이처럼 모든 객체는 `__proto__` 접근자 프로퍼티를 통해 프로토타입을 가리키는 `[[Prototype]]` 내부 슬롯에 접근할 수 있다.

> **`__proto__`는 접근자 프로퍼티다.**

`[[Prototype]]` 내부 슬롯에 직접 접근하는 것은 불가능하므로 `__proto__` 접근자 프로퍼티를 통해 간접적으로 프로토타입에 접근할 수 있게 된다.

> [!NOTE] **접근자 프로퍼티**는 자체적으로 값(`[[Value]]` 프로퍼티 어트리뷰트)을 갖지 않고 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 사용하는 접근자 함수(accessor function)인 `[[Get]]`,`[[Set]]` 프로퍼티 어트리뷰트로 구성된 프로퍼티다.

`Object.prototype`의 접근자 프로퍼티인 `__proto__`는 getter/setter 함수라고 부르는 접근자 함수(`[[Get]]`,`[[Set]]` 프로퍼티 어트리뷰트에 할당된 함수)를 통해 `[[Prototype]]` 내부 슬롯의 값, 즉 프로토타입을 참조하거나 할당하게 된다. `__proto__` 접근자 프로퍼티를 통해 프로토타입에 접근하면 내부적으로 `__proto__` 접근자 프로퍼티의 getter 함수인 `[[Get]]`이 호출되며, 새로운 프로토타입을 할당하면 `__proto__` 접근자 프로퍼티의 setter 함수인 `[[Set]]`이 호출된다.

```javascript
const obj = {};
const parent = { x: 1 };

// getter 함수인 get __proto__가 호출됨
obj.__proto__;
// setter 함수인 set __proto__가 호출됨
obj.__proto__ = parent;

obj.x; // 1
```

> **`__proto__` 접근자 프로퍼티는 상속을 통해 사용된다.**

> [!ATTENTION] `__proto__` 접근자 프로퍼티는 객체가 직접 소유하는 프로퍼티가 아니라 `Object.prototype`의 프로퍼티이며, 모든 객체는 상속을 통해 `Object.prototype.__proto__` 접근자 프로퍼티를 사용할 수 있다.

```javascript
const person = { name: 'Kim' };

// person 객체는 __proto__ 프로퍼티를 소유하지 않는다.
person.hasOwnProperty('__proto__'); // false

// __proto__ 접근자 프로퍼티는 모든 객체의 프로토타입 객체인 Object.prototype의 접근자 프로퍼티이다.
Object.getOwnPropertyDescriptor(Object.prototype, '__proto__');
/*
{
  get: ƒ get __proto__(),
  set: ƒ set __proto__(),
  enumerable: false,
  configurable: true
}
*/

// 모든 객체는 Object.prototype의 접근자 프로퍼티 __proto__를 상속받아 사용하게 된다.
{}.__proto__ === Object.prototype; // true
```

> [!NOTE] **`Object.prototype`**
>
> 모든 객체는 프로토타입의 계층 구조인 프로토타입 체인에 묶여 있다. 자바스크립트 엔진은 객체의 프로퍼티(메서드 포함)에 접근하려 할 때 해당 객체에 접근하려는 프로퍼티가 없다면 `__proto__` 접근자 프로퍼티가 가리키는 참조를 따라 자신의 부모 역할을 하는 프로토타입의 프로퍼티를 순차적으로 검색한다. 프로토타입 체인의 종점(최상위 객체)은 `Object.prototype`이며, 이 객체의 프로퍼티와 메서드는 모든 객체에게 상속된다.

> **`__proto__` 접근자 프로퍼티를 통해 프로토타입에 접근하는 이유**

`[[Prototype]]` 내부 슬롯의 값에 접근하기 위해 접근자 프로퍼티를 사용하는 이유는 상호 참조에 의해 프로토타입 체인이 생성되는 것을 방지하기 위해서이다.

```javascript
const parent = {};
const child = {};

// child의 프로토타입을 parent로 설정
child.__proto__ = parent;

// parent의 프로토타입을 child로 설정
parent..__proto__ = child; // TypeError: Cyclic __proto__ value
```

위의 코드가 에러 없이 정상적으로 처리된다면 서로가 자신의 프로토타입이 되는 비정상적인 프로토타입 체인이 만들어지기 때문에 `__proto__` 접근자 프로퍼티는 에러를 발생시킨다.

프로토타입 체인은 프로퍼티 검색 방향이 한쪽 방향으로만 흘러가도록 Single Linked List로 구현되어야 한다. 따라서 위의 코드처럼 순환 참조(circular reference)하는 프로토타입 체인이 만들어지면 프로토타입 체인 종점이 존재하지 않게 되기 때문에 프로토타입 체인에서 프로퍼티를 검색할 때 무한 루프에 빠지게 된다. 따라서 프로토타입 교체 시 순환 참조에 빠지지 않게 하도록 `__proto__` 접근자 프로퍼티를 통해 프로토타입에 접근하고 교체하도록 구현되어 있는 것이다.

> **`__proto__` 접근자 프로퍼티를 코드 내에서 직접 사용하는 것은 권장하지 않는다.**

ES6에서는 브라우저 호환성을 고려하여 `__proto__`를 표준으로 채택했지만, 코드 내에서 직접 `__proto__` 접근자 프로퍼티를 사용하는 것은 권장되지 않는다. 직접 상속을 통해 다음과 같이 `Object.prototype`을 상속받지 않는 객체를 생성할 수도 있기 때문이다.

```javascript
// obj는 프로토타입 체인의 종점이다. 따라서 Object.__proto__를 상속받을 수 없다.
const obj = Object.create(null);

// obj는 Object.prototype.__proto__를 상속받을 수 없다.
obj.__proto__; // undefined

// 대신 Object.getPrototypeOf 메서드를 사용하자
Object.getPrototypeOf(obj); // null
```

> [!NOTE]
> 프로토타입의 참조를 취득하고 싶은 경우에는 `Object.getPrototypeOf` 메서드, 교체하고 싶은 경우에는 `Object.setPrototypeOf` 메서드를 사용하자.
>
> `Object.getPrototypeOf` → `get Object.prototype.__proto__`
>
> `Object.setPrototypeOf` → `set Object.prototype.__proto__`

### 함수 객체의 `prototype` 프로퍼티

**함수 객체만이 소유하는 `prototype` 프로퍼티는 생성자 함수가 생성할 인스턴스의 프로토타입을 가리킨다.**

```javascript
// 함수 객체는 prototype 프로퍼티를 소유한다.
(function () {}.hasOwnProperty('prototype')); // true

// 일반 객체는 prototype 프로퍼티를 소유하지 않는다.
{}.hasOwnProperty('prototype');
```

`prototype` 프로퍼티는 생성자 함수가 생성할 객체(인스턴스)의 프로토타입을 가리킨다. 따라서 생성자 함수로 호출할 수 없는 non-constructor인 화살표 함수와 ES6 메서드 축약 표현으로 정의한 메서드는 `prototype` 프로퍼티를 소유하지 않으며, 프로토타입 객체도 생성하지 않는다.

```javascript
// 화살표 함수는 non-constructor
const Person = name => {
  this.name = name;
};

// non-constructor는 prototype 프로퍼티를 소유하지 않으며, 프로토타입 객체를 생성하지 않는다.
Person.hasOwnProperty('prototype'); // false
Person.prototype; // undefined

// ES6 메서드 축약 표현으로 정의한 메서드는 non-constructor
const obj = {
  foo() {}
};

obj.foo.hasOwnProperty('prototype'); // false
obj.foo.prototype; // undefined
```

생성자 함수로 호출하기 위해 정의하지 않은 일반 함수(함수 선언문, 함수 표현식)도 prototype 프로퍼티를 소유하지만 객체를 생성하지 않는 일반 함수의 prototype 프로퍼티는 아무런 의미가 없다.

**모든 객체가 가지고 있는(`Object.prototype`으로부터 상속받은) `__proto__` 접근자 프로퍼티와 함수 객체만이 가지고 있는 `prototype` 프로퍼티는 결국 동일한 프로토타입을 가리킨다.** 그러나 각각의 프로퍼티를 사용하는 주체가 다르다.

| 구분                        | 소유             | 값                | 사용 주체   | 사용 목적                                                              |
| --------------------------- | ---------------- | ----------------- | ----------- | ---------------------------------------------------------------------- |
| `__proto__` 접근자 프로퍼티 | 모든 객체        | 프로토타입의 참조 | 모든 객체   | 객체가 자신의 프로토타입에 접근 또는 교체하기 위해 사용                |
| `prototype` 프로퍼티        | constructor only | 프로토타입의 참조 | 생성자 함수 | 생성자 함수가 자신이 생성할 인스턴스의 프로토타입에 할당하기 위해 사용 |

```javascript
// 생성자 함수
function Persono(name) {
  this.name = name;
}

const me = new Person('Kim');

Person.prototype === me.__proto__; // true
```

### 프로토타입의 `constructor` 프로퍼티와 생성자 함수

생성자 함수에 의해 생성된 모든 프로토타입 객체는 `constructor` 프로퍼티를 갖는다. 이때 프로토타입 객체의 `constructor` 프로퍼티는 `prototype` 프로퍼티로 자신을 참조하고 있는 생성자 함수를 가리킨다. **이러한 연결은 생성자 함수 객체가 생성될 때** 이루어진다.

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Kim');

me.constructor === Person; // true
```

<center><img src="docs/javascript/concept/prototype/constructor-relationship.png" alt="constructor-relationship" width="1024" height="768"></center>

`Person` 생성자 함수는 `me` 객체를 생성한 상황에서 `me` 객체는 프로토타입의 `constructor` 프로퍼티를 통해 생성자 함수와 연결된다. `me` 객체에는 `constructor` 프로퍼티가 없지만 `me` 객체의 프로토타입인 `Person.prototype`에는 `constructor` 프로퍼티가 있으므로 `me` 객체는 프로토타입인 `Person.prototype`의 `constructor` 프로퍼티를 상속받아 사용할 수 있게 된다.

## 리터럴 표기법에 의해 생성된 객체의 생성자 함수와 프로토타입

생성자 함수에 의해 생성된 인스턴스는 프로토타입 객체의 `constructor` 프로퍼티에 의해 생성자 함수와 연결된다. 이때 `constructor` 프로퍼티가 가리키는 생성자 함수는 인스턴스를 생성한 생성자 함수이다.

```javascript
// obj 객체를 생성한 생성자 함수는 Object다.
const obj = new Object();
obj.constructor === Object; // true

const add = new Function('a', 'b', 'return a + b');
add.constructor === Function; // true

// 생성자 함수
function Person(name) {
  this.name = name;
}

// me 객체를 생성한 생성자 함수는 Person이다.
const me = new Person('Kim');
me.constructor ==== Person; // true
```

하지만 리터럴 표기법에 의한 객체 생성 방식과 같이 명시적으로 `new` 연산자와 함께 생성자 함수를 호출하여 인스턴스를 생성하지 않는 객체 생성 방식도 있다.

```javascript
// 객체 리터럴
const obj = {};

// 함수 리터럴
const add = function (a, b) {
  return a + b;
};

// 배열 리터럴
const arr = [1, 2, 3];

// 정규표현식 리터럴
const regExp = /is/gi;
```

리터럴 표기법에 의해 생성된 객체 역시 프로토타입이 존재하지만, 리터럴 표기법에 의해 생성된 객체의 경우 프로토타입 객체의 `constructor` 프로퍼티가 가리키는 생성자 함수가 반드시 객체를 생성한 생성자 함수라고 단정할 수는 없다.

```javascript
// obj 객체는 Object 생성자 함수로 생성한 객체가 아니라 객체 리터럴로 생성
const obj = {};

// obj 객체의 생성자 함수는 Object 생성자 함수이다.
obj.constructor === Object; // true
```

객체 리터럴로 생성된 객체도 `Object` 생성자 함수와 `constructor` 프로퍼티로 연결된 것을 보니 객체 리터럴에 의해 생성된 객체 역시 `Object` 생성자 함수로 생성되는 것은 아닐까라는 추측을 해볼 수 있다. ECMAScript 사양에 따르면 `Object` 생성자 함수는 다음과 같이 구현하도록 정의되어 있다.

<center><img src="docs/javascript/concept/prototype/object-constructor.png" alt="object-constructor"></center>

(1)은 `class Foo extends Object {}`와 같이 `Object` 생성자 함수를 확장한 클래스를 호출하는 경우이다.

(2)는 `Object` 생성자 함수에 인수를 전달하지 않거나 `undefined` 또는 `null`을 인수로 전달하면서 `new` 연산자와 함께 호출하면 내부적으로 추상 연산 `OrdinaryObjectCreate`를 호출하여 `Object.prototype`을 프로토타입으로 갖는 빈 객체를 생성한다.

(3)은 `new` 없이 `Object` 생성자 함수를 호출하는 경우이다.

```javascript
// 인수가 전달되지 않은 경우 추상 연산 OrdinaryObjectCreate를 호출하여 빈 객체를 생성한다.
let obj = new Object();
obj; // {}

// new.target이 undefined나 Object가 아닌 경우
// 인스턴스 -> Foo.prototype -> Object.prototype 순으로 프로토타입 체인이 생성된다.
class Foo extends Object {}
new Foo(); // Foo {}

// 인수가 전달된 경우에는 인수를 객체로 변환한다.
// Number 객체 생성
obj = new Object(123);
obj; // Number { 123 }

// String 객체 생성
obj = new Object('123');
obj; // String { '123' }
```

객체 리터럴이 평가될 때는 추상 연산 `OrdinaryObjectCreate`를 호출하여 빈 객체를 생성하고 프로퍼티를 추가하도록 정의되어 있다.

<center><img src="docs/javascript/concept/prototype/object-literal-evaluation.png" alt="object-literal-evalutation"></center>

이처럼 `Object` 생성자 함수 호출과 객체 리터럴의 평가는 추상 연산 `OrdinaryObjectCreate`를 호출하여 빈 객체를 생성한다는 점에서는 동일하나 `new.target`의 확인이나 프로퍼티를 추가하는 처리 등 세부 내용은 다르다. 따라서 객체 리터럴에 의해 생성된 객체는 `Object` 생성자 함수가 생성한 객체가 아니다.

함수 객체의 경우 `Function` 생성자 함수를 호출하여 생성한 함수는 렉시컬 스코프를 만들지 않고 전역 함수인 것처럼 스코프를 생성하며 클로저도 만들지 않는다. 따라서 함수 선언문과 함수 표현식을 평가하여 함수 객체를 생성한 것은 `Function` 생성자 함수가 아니다. 하지만 `constructor` 프로퍼티를 통해 확인해보면 `foo` 함수의 생성자 함수는 `Function` 생성자 함수로 나온다.

```javascript
// foo 함수는 Function 생성자 함수로 생성한 함수 객체가 아니라 함수 선언문으로 생성했다.
function foo() {}

// but, constructor 프로퍼티를 확인해보면 Function 생성자 함수로 나온다.
foo.constructor === Function; // true
```

리터럴 표기법에 의해 생성된 객체도 상속을 위해 프로토타입이 필요하다. 따라서 리터럴 표기법에 의해 생성된 객체도 가상적인 생성자 함수를 갖게 된다. 프로토타입 객체는 생성자 함수 객체가 평가될 때 생성되며 `prototype` 프로퍼티와 `constructor` 프로퍼티로 서로 연결되어 있기 때문이다. 따라서 **프로토타입 객체와 생성자 함수는 단독으로 존재할 수 없고 언제나 쌍으로 존재한다.**

리터럴 표기법에 의해 생성된 객체는 생성자 함수에 의해 생성된 객체는 아니지만, 큰 틀에서 생각해보면 리터럴 표기법으로 생성한 객체 역시 생성자 함수로 생성한 객체와 본질적으로 큰 차이는 없다.

객체 리터럴의 경우에도 `Object` 생성자 함수에 의해 생성한 객체와 생성 과정에 미묘한 차이는 있지만 객체로서 동일한 특성을 갖고, 함수 리터럴에 의해 생성한 함수도 `Function` 생성자 함수에 의해 생성된 함수와 생성 과정, 스코프, 클로저 등의 차이는 있지만 결국 함수로서 동일한 특성을 갖기 때문이다.

> [!NOTE]
> 프로토타입 객체의 `constructor` 프로퍼티를 통해 연결되어 있는 생성자 함수를 리터럴 표기법으로 생성한 객체를 생성한 생성자 함수로 생각해도 큰 무리는 없다.

## 프로토타입의 생성 시점

객체는 리터럴 표기법 또는 생성자 함수에 의해 생성되므로 결국 모든 객체는 생성자 함수와 연결되어 있다.

**프로토타입 객체는 생성자 함수가 생성되는 시점**에 생성되며, 이때 생성자 함수는 사용자가 직접 정의한 사용자 정의 생성자 함수와 자바스크립트가 기본적으로 제공하는 빌트인 생성자 함수로 구분할 수 있다.

### 사용자 정의 생성자 함수의 경우

내부 메서드 `[[Construct]]`를 갖는 일반 함수는 `new` 연산자와 함께 생성자 함수로서 호출할 수 있다.

**생성자 함수로서 호출할 수 있는 함수, 즉 constructor는 함수 정의가 평가되어 함수 객체를 생성하는 시점에 프로토타입 객체도 생성된다.**

```javascript
Person.prototype; // {constructor: ƒ}

// 생성자 함수
function Person(name) {
  this.name = name;
}
```

이때 생성자 함수로서 호출할 수 없는 non-constructor는 프로토타입이 생성되지 않는다.

```javascript
const Person = name => {
  this.name = name;
};

Person.prototype; // undefined
```

생성자 함수가 평가될 때 생성된 프로토타입은 오직 `constructor` 프로퍼티만을 갖는 객체이다. 프로토타입 객체 역시 객체이고 모든 객체는 프로토타입을 가지므로 프로토타입 객체 역시 자신의 프로토타입을 갖게 된다. 이때 생성된 프로토타입 객체의 프로토타입은 `Object.prototype`이다.

<center><img src="docs/javascript/concept/prototype/person.prototype.png" alt="person.prototype" width="1024" height="768"></center>

이처럼 빌트인 생성자 함수가 아닌 사용자 정의 생성자 함수는 자신이 평가되어 함수 객체로 생성되는 시점에 프로토타입 객체 역시 생성하며, 이때 생성된 프로토타입 객체의 프로토타입은 언제나 `Object.prototype`이다.

### 빌트인 생성자 함수의 경우

`Object`, `String`, `Number`, `Function`, `Array`, `RegExp`, `Date`, `Promise` 등과 같은 빌트인 생성자 함수도 일반 함수와 마찬가지로 빌트인 생성자 함수가 생성되는 시점에 프로토타입 객체를 생성한다. 모든 빌트인 생성자 함수는 전역 객체가 생성되는 시점에 생성되며, 생성된 프로토타입 객체는 빌트인 생성자 함수의 `prototype` 프로퍼티에 바인딩된다.

> [!NOTE] **전역 객체**(**global object**)
>
> 전역 객체는 코드가 실행되기 이전 단계에 자바스크립트 엔진에 의해 생성되는 특수한 객체다. 전역 객체는 브라우저에서는 `window`, Node.js에서는 `global` 객체를 의미한다.
>
> 전역 객체는 표준 빌트인 객체(`Object`,`String`,`Number`,`Function`,`Array` 등)들과 환경에 따른 호스트 객체(클라이언트 web API 또는 Node.js의 호스트 API), 그리고 `var` 키워드로 선언한 전역 변수와 전역 함수를 프로퍼티로 갖는다. `Math`,`Reflect`,`JSON`을 제외한 표준 빌트인 객체는 모두 생성자 함수이다.

```javascript
// 빌트인 객체인 Object는 전역 객체 window의 프로퍼티다.
window.Object === Object; // true
```

표준 빌트인 객체인 `Object`도 전역 객체의 프로퍼티이며, 전역 객체가 생성되는 시점에 생성된다.

이처럼 객체가 생성되기 이전에 생성자 함수와 프로토타입 객체는 이미 객체화되어 존재한다. **이후 생성자 함수 또는 리터럴 표기법으로 객체를 생성하면 프로토타입 객체는 생성된 객체의 `[[Prototype]]` 내부 슬롯에 할당된다.** 이로써 생성된 객체는 프로토타입을 상속받게 된다.

## 객체 생성 방식과 프로토타입의 결정

> **객체 생성 방법**

- 객체 리터럴
- `Object` 생성자 함수
- 생성자 함수
- `Object.create` 메서드
- 클래스 (ES6)

다양한 방식으로 생성된 모든 객체는 세부적인 생성 방식의 차이는 있으나 추상 연산 `OrdinaryObjectCreate`에 의해 생성된다는 점은 동일하다.

추상 연산 `OrdinaryObjectCreate`는 필수적으로 자신이 생성할 객체의 프로토타입을 인수로 전달받는다. 그리고 자신이 생성할 객체에 추가할 프로퍼티 목록을 옵션으로 전달할 수 있다. 추상 연산 `OrdinaryObjectCreate`는 빈 객체를 생성한 후, 객체에 추가할 프로퍼티 목록이 인수로 전달된 경우 프로퍼티를 객체에 추가한다. 그리고 인수로 잔달받은 프로토타입 객체를 자신이 생성한 객체의 `[[Prototype]]` 내부 슬롯에 할당한 다음, 생성한 객체를 반환한다.

> [!NOTE]
> 프로토타입은 추상 연산 `OrdinaryObjectCreate`에 전달되는 인수에 의해 결정되며, 이 인수는 객체가 생성되는 시점에 **객체 생성 방식**에 의해 결정된다.

### 객체 리터럴에 의해 생성된 객체의 프로토타입

자바스크립트 엔진은 객체 리터럴을 평가하여 객체를 생성할 때 추상 연산 `OrdinaryObjectCreate`를 호출한다. 이때 인수로 전달되는 프로토타입 객체는 `Object.prototype`이다. 따라서 객체 리터럴에 의해 생성되는 객체의 프로토타입은 `Object.prototype`이다.

```javascript
const obj = { x: 1 };
```

<center><img src="docs/javascript/concept/prototype/object-literal.png" alt="object-literal" width="1024" height="768"></center>

객체 리터럴에 의해 생성된 `obj` 객체는 `Object.prototype`을 프로토타입으로 갖게 되며, 이로써 `Object.prototype`을 상속받게 된다. 따라서 `obj` 객체는 `constructor` 프로퍼티와 `hasOwnProperty` 메서드 등을 소유하지는 않지만 자신의 프로토타입인 `Object.prototype`의 `constructor` 프로퍼티와 `hasOwnProperty` 메서드를 자신의 프로퍼티인 것처럼 사용할 수 있게 되는 것이다.

```javascript
const obj = { x: 1 };

// 객체 리터럴에 의해 생성된 obj 객체는 Object.prototype을 상속받는다.
obj.constructor === Object; // true
obj.hasOwnProperty('x'); // true
```

### `Object` 생성자 함수에 의해 생성된 객체의 프로토타입

`Object` 생성자 함수를 인수 없이 호출하면 빈 객체가 생성된다. `Object` 생성자 함수도 객체 리터럴과 마찬가지로 추상 연산 `OrdinaryObjectCreate`가 호출된다. 이때 인수로 전달되는 프로토타입 객체는 `Object.prototype`이다. 따라서 `Object` 생성자 함수에 의해 생성되는 객체의 프로토타입은 `Object.prototype`이다.

```javascript
const obj = new Object();
obj.x = 1;
```

<center><img src="docs/javascript/concept/prototype/object-constructor-prototype.png" alt="object-constructor-prototype" width="1024" height="768"></center>

따라서 `Object` 생성자 함수에 의해 생성된 `obj` 객체는 `Object.prototype`을 프로토타입으로 갖게 되며, `Object.prototype`을 상속받는다.

```javascript
const obj = new Object();
obj.x = 1;

obj.constructor === Object; // true
obj.hasOwnProperty('x'); // true
```

객체 리터럴과 `Object` 생성자 함수에 의한 객체 생성 방식의 차이는 프로퍼티를 추가하는 방식에 있다. 객체 리터럴은 리터럴 내부에 프로퍼티를 추가하지만 생성자 함수 방식은 일단 빈 객체를 생성한 이후 프로퍼티를 추가해야 한다.

### 생성자 함수에 의해 생성된 객체의 프로토타입

`new` 연산자와 함께 생성자 함수를 호출하여 인스턴스를 생성하면 다른 객체 생성 방식과 마찬가지로 추상 연산 `OrdinaryObjectCreate`가 호출된다. 이때 추상 연산 `OrdinaryObjectCreate`에 전달되는 프로토타입은 생성자 함수의 `prototype` 프로퍼티에 바인딩되어 있는 객체다. 즉, 생성자 함수에 의해 생성되는 객체의 프로토타입은 생성자 함수의 `prototype` 프로퍼티에 바인딩되어 있는 객체다.

```javascript
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');
```

위 코드가 실행되면 추상 연산 `OrdinaryObjectCreate`에 의해 생성자 함수와 생성자 함수의 `prototype` 프로퍼티에 바인딩되어 있는 객체와 생성된 객체 사이에 연결이 만들어 진다.

표준 빌트인 객체인 `Object` 생성자 함수와 더불어 생성된 프로토타입 객체 `Object.prototype`은 다양한 빌트인 메서드(`hasOwnProperty`,`propertyIsEnumerable` 등)를 갖고 있다. 그러나 사용자 정의 생성자 함수 `Person`과 생성된 프로토타입 객체 `Person.prototype`의 프로퍼티는 `constructor` 뿐이다.

<center><img src="docs/javascript/concept/prototype/constructor-relationship.png" alt="constructor-relationship" width="1024" height="768"></center>

프로토타입 객체 `Person.prototype`에 프로퍼티를 추가하여 자식 객체가 상속받을 수 있도록 구현해보자. 프로토타입 객체 역시 객체이므로 일반 객체와 같이 프로토타입 객체에도 프로퍼티를 추가/삭제하는 것이 가능하다. 그리고 이렇게 추가/삭제된 프로퍼티는 프로토타입 체인에 즉각 반영된다.

```javascript
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

const me = new Person('Kim');
const you = new Person('Lee');

me.sayHello(); // Hi! My name is Kim
you.sayHello(); // Hi! My name is Lee
```

<center><img src="docs/javascript/concept/prototype/method-inheritance.png" alt="method-inheritance" width="1024" height="768"></center>

## 프로토타입 체인

```javascript
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

const me = new Person('Kim');

// hasOwnProperty는 Object.prototype의 메서드다
me.hasOwnProperty('name'); // true
```

`Person` 생성자 함수에 의해 생성된 `me` 객체가 `Object.prototype`의 메서드인 `hasOwnProperty`를 호출할 수 있다. 이는 `me` 객체가 `Person.prototype` 뿐만 아니라 `Object.prototype`도 상속받았음을 의미한다.

```javascript
Object.getPrototypeOf(me) === Person.prototype; // true
```

`Person.prototype`의 프로토타입은 `Object.prototype`이며, 프로토타입 객체의 프로토타입은 언제나 `Object.prototype`이다.

```javascript
Object.getPrototypeOf(Person.prototype) === Object.prototype; // true
```

<center><img src="https://poiemaweb.com/assets/fs-images/19-18.png" alt="prototype-chain" width="1024" height="768"></center>

> [!NOTE] **프로토타입 체인**
>
> 자바스크립트는 객체의 프로퍼티(메서드 포함)에 접근하려고 할 때 해당 객체에 접근하려는 프로퍼티가 존재하지 않는다면 `[[Prototype]]` 내부 슬롯의 참조를 따라 자신의 부모 역할을 하는 프로토타입의 프로퍼티를 순차적으로 검색하며 이를 프로토타입 체인이라 한다. 프로토타입 체인은 자바스크립트가 객체지향 프로그래밍의 상속을 구현하는 메커니즘이다.

```javascript
// hasOwnProperty는 Object.prototype의 메서드다.
// me 객체는 프로토타입 체인을 따라 hasOwnProperty 메서드를 검색하여 사용한다.
me.hasOwnProperty('name'); // true
```

> [!ATTENTION]
> 프로토타입 체인에서 `hasOwnProperty` 메서드를 찾아 호출할 때 자바스크립트 엔진은 `call` 메서드를 이용해 `this`로 사용할 객체를 전달하면서 호출하게 된다. 이 경우에는 `me` 객체가 `this`로 바인딩되게 되며 다음과 같이 사용된다.
>
> ```javascript
> Object.prototype.hasOwnProperty.call(me, 'name');
> ```

프로토타입 체인의 최상위에 위치하는 객체는 언제나 `Object.prototype`이며 따라서 모든 객체는 `Object.prototype`을 상속받는다. 이때 `Object.prototype`의 프로토타입(`[[Prototype]]` 내부 슬롯)의 값은 `null`이다.

> [!ATTENTION]
> 이때 프로토타입 체인의 종점인 `Object.prototype`에서도 프로퍼티를 검색할 수 없는 경우 에러가 아닌 `undefined`를 반환한다는 것에 주의하자.

```javascript
me.foo; //undefined
```

> **프로토타입 체인과 스코프 체인 간의 관계**

자바스크립트 엔진은 프로토타입 체인을 따라 프로퍼티/메서드를 검색하므로 객체 간의 상속 관계로 이루어진 프로토타입의 계층적 구조를 이용하여 객체의 프로퍼티를 검색한다.

이에 반해 식별자는 스코프 체인에서 검색하는데, 함수의 중첩 관계로 이루어진 스코프의 계층적 구조를 이용하여 식별자를 검색하게 된다.

```javascript
me.hasOwnProperty('name');
```

위의 코드에서 먼저 스코프 체인에서 `me` 식별자를 검색하게 된다. 이때 식별자 `me`는 전역에서 선언되었으므로 전역 스코프에서 검색된다. 그 다음 `me` 객체의 프로토타입 체인에서 `hasOwnProperty` 메서드를 검색하게 된다.

## 오버라이딩과 프로퍼티 섀도잉

```javascript
const Person = (function () {
  // 생성자 함수
  function Person(name) {
    this.name = name;
  }

  // 프로토타입 메서드
  Person.prototype.sayHello = function () {
    console.log(`Hi! My name is ${this.name}`);
  };

  // 생성자 함수를 반환
  return Person;
})();

const me = new Person('Kim');

// 인스턴스 메서드
me.sayHello = function () {
  console.log(`Hey! My name is ${this.name}`);
};

// 인스턴스 메서드가 호출된다. 프로토타입 메서드는 인스턴스 메서드에 의해 가려진다.
me.sayHello(); // Hey! My name is Kim
```

위의 경우에서 프로토타입이 소유한 프로퍼티(메서드 포함)를 프로토타입 프로퍼티, 인스턴스가 소유한 프로퍼티를 인스턴스 프로퍼티라 한다.

프로토타입 프로퍼티와 같은 이름의 프로퍼티를 인스턴스에 추가하면 프로토타입 체인을 따라 프로토타입 프로퍼티를 검색하여 프로토타입 프로퍼티를 덮어쓰는 것이 아니라 인스턴스 프로퍼티로 추가한다. 이때 인스턴스 메서드 `sayHello`는 프로토타입 메서드 `sayHello`를 오버라이딩하였고 따라서 프로토타입 메서드는 가려지게 된다. 이처럼 상속 관계에 의해 프로퍼티가 가려지는 현상을 **프로퍼티 섀도잉**(**property shadowing**)이라 한다.

> [!NOTE] **오버라이딩**
>
> 상위 클래스가 가지고 있는 메서드를 하위 클래스가 재정의하여 사용하는 방식
>
> **오버로딩**
>
> 함수명은 동일하지만 매개변수의 타입 또는 개수가 다른 메서드를 구현하고 매개변수에 의해 메서드를 구별하여 호출하는 방식으로 자바스크립트는 오버로딩을 지원하지 않지만 `arguments` 객체를 사용하여 구현은 가능하다.

프로퍼티 삭제의 경우도 마찬가지이다.

```javascript
// 인스턴스 메서드를 삭제한다.
delete me.sayHello;
// 인스턴스의 프로퍼티에는 sayHello 메서드가 없으므로 프로토타입 메서드가 호출된다.
me.sayHello(); // Hi! My name is Kim
```

단 하위 객체를 통해 프로토타입의 프로퍼티를 변경 또는 삭제하는 것은 불가능하며, 이는 하위 객체에서는 프로토타입에 get 액세스만 허용한다는 것을 의미한다.

따라서 프로토타입 프로퍼티를 변경 또는 삭제하려면 프로토타입에 직접 접근해야 한다.

```javascript
// 프로토타입 메서드 변경
Person.prototype.sayHello = function () {
  console.log(`Hey! My name is ${this.name}`);
};
me.sayHello(); // Hey! My name is Kim

// 프로토타입 메서드 삭제
delete Person.prototype.sayHello;
me.sayHello(); // TypeError: me.sayHello is not a function
```

## 프로토타입의 교체

프로토타입은 임의의 다른 객체로 변경할 수 있다. 이는 부모 객체인 프로토타입 객체를 동적으로 변경할 수 있다는 것을 의미하며 이러한 특성을 활용해 객체 간의 상속 관계를 동적으로 변경할 수 있다. 프로토타입 객체는 생성자 함수 또는 인스턴스에 의해 교체할 수 있다.

### 생성자 함수에 의한 프로토타입의 교체

```javascript
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  // ① 생성자 함수의 prototype 프로퍼티를 통해 프로토타입을 교체
  Person.prototype = {
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    }
  };

  return Person;
})();

const me = new Person('Kim');
```

①에서 `Person.prototype`에 객체 리터럴을 할당하였다. 이는 `Person` 생성자 함수가 생성할 객체의 프로토타입을 객체 리터럴로 교체한 것이다.

<center><img src="https://poiemaweb.com/assets/fs-images/19-20.png" alt="prototype-replace-by-constructor" width="1024" height="768"></center>

새롭게 프로토타입 객체로 교체한 객체 리터럴에는 `constructor` 프로퍼티가 없으므로 `me` 객체의 생성자 함수를 검색하면 `Person`이 아닌 `Object`가 나온다.

```javascript
me.constructor === Person; // false
me.constructor === Object; // true
```

이처럼 프로토타입을 교체하면 `constructor` 프로퍼티와 생성자 함수 간의 연결이 파괴된다. 파괴된 `constructor` 프로퍼티와 생성자 함수 간의 연결을 되살리려면 객체 리터럴에 `constructor` 프로퍼티를 추가하면 된다.

```javascript
const Person = (function () {
  function Person(name) {
    this.name = name;
  }

  // 생성자 함수의 prototype 프로퍼티를 통해 프로토타입을 교체
  Person.prototype = {
    constructor: Person,
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    }
  };

  return Person;
})();

const me = new Person('Lee');

me.constructor === Person; // true
me.constructor === Object; // false
```

### 인스턴스에 의한 프로토타입의 교체

프로토타입은 생성자 함수의 `prototype` 프로퍼티뿐만 아니라 인스턴스의 `__proto__` 접근자 프로퍼티(혹은 `Object.getPrototypeOf` 메서드)를 통해 접근할 수 있다. 따라서 인스턴스의 `__proto__` 접근자 프로퍼티(혹은 `Object.setPrototypeOf` 메서드)를 통해 프로토타입을 교체할 수 있다.

생성자 함수의 `prototype` 프로퍼티에 다른 임의의 객체를 바인딩하는 것은 미래에 생성할 인스턴스의 프로토타입을 교체하는 것이다. `__proto__` 접근자 프로퍼티를 통해 프로토타입을 교체하는 것은 이미 생성된 객체의 프로토타입을 교체하는 것이다.

```javascript
function Person(name) {
  this.name = name;
}

const me = new Person('Kim');

// 프로토타입 객체로 교체할 객체
const parent = {
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  }
};

// ① me 객체의 프로토타입을 parent 객체로 교체한다.
Object.setPrototypeOf(me, parent);
// me.__proto__ = parent;

me.sayHello(); // Hi! My name is Kim
```

<center><img src="https://poiemaweb.com/assets/fs-images/19-21.png" alt="prototype-replace-with-instance" width="1024" height="768"></center>

이때 프로토타입 객체로 교체한 객체에 `constructor` 프로퍼티가 없으므로 생성자 함수를 참조할 수 있는 방법이 사라진다. 따라서 프로토타입의 `constructor` 프로퍼티로 `me` 객체의 생성자 함수를 검색하면 `Object`가 된다.

> [!NOTE]
> 생성자 함수에 의한 프로토타입 교체와 인스턴스에 의한 프로토타입 교체는 미묘한 차이가 존재한다.
>
> <center><img src="https://poiemaweb.com/assets/fs-images/19-22.png" alt="comparison" width="1024" height="768"></center>
>
> 이는 인스턴스에 의한 프로토타입 교체의 경우에는 `Person` 생성자 함수의 `prototype` 프로퍼티와 `parent` 객체를 연결해주는 과정이 없기 때문이다.

```javascript
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

const parent = {
  constructor: Person,
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  }
};

// 생성자 함수의 prototype 프로퍼티와 프로토타입 간의 연결을 설정
Person.prototype = parent;

Object.setPrototypeOf(me, parent);

me.sayHello(); // Hi! My name is Lee

me.constructor === Person; // true
me.constructor === Object; // false

Person.prototype === Object.getPrototypeOf(me); // true
```

프로토타입 교체를 통해 객체 간의 상속 관계를 동적으로 변경하는 것은 꽤나 번거로운 작업이다. 하지만 ES6에서 도입된 클래스를 사용하면 간편하고 직관적으로 상속 관계를 구현할 수 있다.

> [!TIP]
> 상속 관계를 인위적으로 설정하려면 직접 상속이 더 편리하고 안전하다. 혹은 ES6에 도입된 클래스를 사용하여 상속 관계를 구현하는 것이 낫다.

## instanceof 연산자

`instanceof` 연산자는 이항 연산자로 좌변에는 객체를 가리키는 식별자, 우변에는 생성자 함수를 가리키는 식별자를 피연산자로 받는다. 이때 우변의 피연산자가 함수가 아닌 경우 `TypeError`가 발생한다.

```javascript
객체 instanceof 생성자 함수
```

**우변의 생성자 함수의 `prototype` 프로퍼티에 바인딩된 객체가 좌변의 객체의 프로토타입 체인 상에 존재하면 `true`, 그렇지 않으면 `false`로 평가된다.**

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

// Person.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true
me instanceof Person; // true

// Object.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true
me instanceof Object; // true
```

그렇다면 인스턴스에 의해 프로토타입을 교체한 경우 `instanceof` 연산자가 어떻게 동작하는지 살펴보자.

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

// 프로토타입으로 교체할 객체
const parent = {};

// 프로토타입의 교체
Object.setPrototypeOf(me, parent);

// Person 생성자 함수와 parent 객체는 연결되어 있지 않다.
Person.prototype === parent; // false
parent.constructor === Person; // false

// Person.prototype이 me 객체의 프로토타입 체인 상에 존재하지 않기 때문에 false
me instanceof Person; // false

// Object.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true
me instanceof Object; // true
```

`me` 객체는 프로토타입이 교체되어 프로토타입과 생성자 함수 간의 연결은 파괴되었지만 `Person` 생성자 함수에 의해 생성된 인스턴스임에는 틀림없다. 하지만 `me instanceof Person`은 `false`로 평가된다.

이는 `Person.prototype`이 `me` 객체의 프로토타입 체인 상에 존재하지 않기 때문이다. 따라서 프로토타입으로 교체한 `parent` 객체를 `Person` 생성자 함수의 `prototype` 프로퍼티에 바인딩하면 `me instanceof Person`은 `true`로 평가될 것이다.

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

const parent = {};

Object.setPrototypeOf(me, parent);
Person.prototype = parent;

me instanceof Person; // true
me instanceof Object; // true
```

이처럼 `instanceof` 연산자는 프로토타입의 `constructor` 프로퍼티가 가리키는 생성자 함수를 찾는 것이 아니라 **생성자 함수의 `prototype`에 바인딩된 객체가 프로토타입 체인 상에 존재하는지 확인한다.**

<center><img src="https://poiemaweb.com/assets/fs-images/19-23.png" alt="instanceof" width="1024" height="768"></center>

```javascript
function isInstanceof(instance, constructor) {
  const prototype = Object.getPrototypeOf(instnace);

  // prototype이 null이면 프로토타입 체인의 종점에 다다른 것이다.
  if (prototype === null) return false;

  // 프로토타입이 생성자 함수의 prototype 프로퍼티에 바인딩된 객체라면 true를 반환하고, 그렇지 않은 경우 재귀 호출로 프로토타입 체인 상의 상위 프로토타입으로 이동하여 확인한다.
  return (
    prototype === constructor.prototype || isInstanceof(prototype, constructor)
  );
}

isInstanceof(me, Person); // true
isInstanceof(me, Object); // true
isInstanceof(me, Array); // false
```

따라서 생성자 함수에 의해 프로토타입이 교체되어 `constructor` 프로퍼티와 생성자 함수 간의 연결이 파괴되어도 생성자 함수의 `prototype` 프로퍼티와 프로토타입 간의 연결은 파괴되지 않으므로 `instanceof`는 아무런 영향을 받지 않는다.

## 직접 상속

### Object.create에 의한 직접 상속

`Object.create` 메서드는 명시적으로 프로토타입을 지정하여 새로운 객체를 생성한다. `Object.create` 메서드도 다른 객체 생성 방식과 마찬가지로 추상 연산 `OrdinaryObjectCreate`를 호출한다.

`Object.create` 메서드의 첫 번째 매개변수에는 생성할 객체의 프로토타입으로 지정할 객체를 전달한다. 두 번째 매개변수에는 생성할 객체의 프로퍼티 키와 프로퍼티 디스크립터 객체로 이뤄진 객체를 전달한다. 이 객체의 형식은 `Object.defineProperties` 메서드의 두 번째 인수와 동일하며, 옵션이므로 생략 가능하다.

```javascript
/**
 * 지정된 프로토타입 및 프로퍼티를 갖는 새로운 객체를 생성하여 반환한다.
 * @param {Object} prototype - 생성할 객체의 프로토타입으로 지정할 객체
 * @param {Object} [propertiesObject] - 생성할 객체의 프로퍼티를 갖는 객체
 * @returns {Object} 지정된 프로토타입 및 프로퍼티를 갖는 새로운 객체
 */
Object.create(prototype[, propertiesObject]);
```

```javascript
// 프로토타입이 null인 객체를 생성한다. 생성된 객체는 프로토타입 체인의 종점에 위치한다.
// obj -> null
let obj = Object.create(null);
Object.getPrototypeOf(obj) === null; // true
// Object.prototype을 상속받지 못한다.
obj.toString(); // TypeError: obj.toString is not a function

// obj -> Object.prototype -> null
// obj = {};와 동일하다.
obj = Object.create(Object.prototype);
Object.getPrototypeOf(obj) === Object.prototype; // true

// obj -> Object.prototype -> null
// obj = { x : 1 };와 동일하다.
obj = Object.create(Object.prototype, {
  x: { value: 1, writable: true, enumerable: true, configurable: true }
});

// 위의 코드는 다음과 동일하다
// obj = Object.create(Object.prototype);
// obj.x = 1;
obj.x; // 1
Object.getPrototypeOf(obj) === Object.prototype; // true

const myProto = { x: 10 };
// 임의의 객체를 직접 상속받는다.
// obj -> myProto -> Object.prototype -> null
obj = Object.create(myProto);
obj.x; // 10
Object.getPrototypeOf(obj) === myProto; // true

// 생성자 함수
function Person(name) {
  this.name = name;
}

// obj -> Person.prototype -> Object.prototype -> null
// obj = new Person('Lee')와 동일하다.
obj = Object.create(Person.prototype);
obj.name = 'Lee';
obj.name; // Lee
Object.getPrototypeOf(obj) === Person.prototype; // true
```

> **`Object.create` 메서드의 장점**

- `new` 연산자가 없이도 객체를 생성할 수 있다.
- 프로토타입을 지정하면서 객체를 생성할 수 있다.
- 객체 리터럴에 의해 생성된 객체도 상속받을 수 있다.

> [!NOTE] `Object.prototype`의 빌트인 메서드인 `Object.prototype.hasOwnProperty`, `Object.prototype.isPrototypeOf`, `Object.prototype.propertyIsEnumerable` 등은 모든 객체의 프로토타입 체인의 종점, `Object.prototype`의 메서드이므로 모든 객체가 상속받아 호출할 수 있다.

ESLint에서는 `Object.prototype`의 빌트인 메서드를 객체가 직접 호출하는 것을 권장하지 않는다. 그 이유는 `Object.create` 메서드를 통해 프로토타입 체인의 종점에 위치하는 객체를 생성할 수 있기 때문이다. 프로토타입 체인의 종점에 위치하는 객체는 `Object.prototype`의 빌트인 메서드를 사용할 수 없다.

따라서 `Object.prototype`의 빌트인 메서드는 다음과 같이 간접적으로 호출하는 것이 좋다.

```javascript
// 프로토타입이 null인 객체를 생성한다.
const obj = Object.create(null);
obj.a = 1;

obj.hasOwnProperty('a'); // TypeError: obj.hasOwnProperty is not a function

// Object.prototype의 빌트인 메서드는 객체로 직접 호출하지 않는다.
Object.prototype.hasOwnProperty.call(obj, 'a'); // true
```

### 객체 리터럴 내부에서 `__proto__`에 의한 직접 상속

ES6에서는 객체 리터럴 내부에서 `__proto__` 접근자 프로퍼티를 사용하여 직접 상속을 구현할 수 있다.

```javascript
const myProto = { x: 10 };

// 객체 리터럴로 객체를 생성하면서 프로토타입을 지정하여 직접 상속받을 수 있다.
const obj = {
  y: 20,
  // 객체를 직접 상속받는다.
  // obj -> myProto -> Object.prototype -> null
  __proto__: myProto
};

/*
const obj = Object.create(myProto, {
  y: { value: 20, writable: true, enumerable: true, configurable: true }
})
*/

obj.x; // 10
obj.y; // 20
Object.getPrototypeOf(obj) === myProto; // true
```

## 정적 프로퍼티/메서드

정적(static) 프로퍼티/메서드는 생성자 함수로 인스턴스를 생성하지 않아도 참조/호출할 수 있는 프로퍼티/메서드를 말한다.

```javascript
// 생성자 함수
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

// 정적 프로퍼티
Person.staticProp = 'static prop';

// 정적 메서드
Person.staticMethod = function () {
  console.log('staticMethod');
};

const me = new Persono('Lee');

// 생성자 함수에 추가한 정적 프로퍼티/메서드는 생성자 함수로 참조/호출한다.
Person.staticMethod(); // staticMethod

// 정적 프로퍼티/메서드는 생성자 함수가 생성한 인스턴스로 참조/호출할 수 없다.
// 인스턴스로 참조/호출할 수 있는 프로퍼티/메서드는 프로토타입 체인 상에 존재해야 한다.
me.staticMethod(); // TypeError: me.staticMethod is not a function
```

<center><img src="https://poiemaweb.com/assets/fs-images/19-24.png" alt="static" width="1024" height="768"></center>

앞서 살펴본 `Object.create` 메서드는 `Object` 생성자 함수의 정적 메서드고 `Object.prototype.hasOwnProperty` 메서드는 `Object.prototype`의 메서드다. 따라서 `Object.create` 메서드는 인스턴스에서 호출하는 것이 불가능하다. 반면 `Object.prototype.hasOwnProperty` 메서드는 모든 객체의 프로토타입 체인의 종점 `Object.prototype`의 메서드이므로 모든 객체가 호출할 수 있다.

만약 인스턴스/프로토타입 메서드 내에서 `this`를 사용하지 않는다면 그 메서드는 정적 메서드로 변경할 수 있다. 인스턴스가 호출한 인스턴스/프로토타입 메서드 내에서 `this`는 인스턴스를 가리킨다. 메서드 내에서 인스턴스를 참조할 필요가 없다면 정적 메서드로 변경해도 동작한다. 프로토타입 메서드를 호출하려면 인스턴스를 생성해야 하지만 정적 메서드는 인스턴스를 생성하지 않아도 호출할 수 있다.

```javascript
function Foo() {}

// 프로토타입 메서드
// this를 참조하지 않는 프로토타입 메서드는 정적 메서드로 변경해도 동일한 효과를 얻을 수 있다.
Foo.prototype.x = function () {
  console.log('x');
};

const foo = new Foo();
// 프로토타입 메서드를 호출하려면 인스턴스를 생성해야 한다.
foo.x(); // x

// 정적 메서드
Foo.x = function () {
  console.log('x');
};

// 정적 메서드는 인스턴스를 생성하지 않아도 호출할 수 있다.
Foo.x(); // x
```

## 프로퍼티 존재 확인

### in 연산자

`in` 연산자는 객체 내에 특정 프로퍼티가 존재하는 지 여부를 확인한다.

```javascript
/**
 * key: 프로퍼티 키를 나타내는 문자열
 * object: 객체로 평가되는 표현식
 */
key in object;
```

```javascript
const person = {
  name: 'Lee',
  address: 'Seoul'
};

// person 객체에 name 프로퍼티가 존재한다.
'name' in person; // true
// person 객체에 address 프로퍼티가 존재한다.
'address' in person; // true
// person 객체에 age 프로퍼티가 존재하지 않는다.
'age' in person; // false
```

> [!ATTENTION] `in` 연산자는 확인 대상 객체(위에서는 `person` 객체)의 프로퍼티뿐만 아니라 확인 대상 객체가 상속받은 모든 프로토타입의 프로퍼티를 확인하므로 주의가 필요하다.
>
> ```javascript
> 'toString' in person; // true
> ```

`in` 연산자 대신 ES6에서 도입된 `Reflect.has` 메서드를 사용할 수도 있다.

```javascript
const person = { name: 'Lee' };
Reflect.has(person, 'name'); // true
Reflect.has(person, 'toString'); // true
```

### Object.prototype.hasOwnProperty 메서드

`Object.prototype.hasOwnProperty` 메서드는 인수로 전달받은 프로퍼티 키가 객체 고유의 프로퍼티 키인 경우에만 `true`를 반환하고 상속받은 프로토타입의 프로퍼티 키인 경우 `false`를 반환한다.

```javascript
person.hasOwnProperty('name'); // true
person.hasOwnProperty('age'); // false
person.hasOwnProperty('toString'); // false
```

## 프로퍼티 열거

### for...in 문

객체의 모든 프로퍼티를 순회하며 열거할 때 사용한다.

```javascript
const person = {
  name: 'Lee',
  address: 'Seoul'
};

// for...in 문의 변수 prop에 person 객체의 프로퍼티 키가 할당된다.
for (const key in person) {
  console.log(key + ': ' + person[key]);
}
// name: Lee
// address: Seoul
```

for...in 문은 `in` 연산자처럼 순회 대상 객체의 프로퍼티 뿐만 아니라 상속받은 프로토타입의 프로퍼티까지 열거한다.

```javascript
const person = {
  name: 'Lee',
  address: 'Seoul'
};

// in 연산자는 객체가 상속받은 모든 프로토타입의 프로퍼티를 확인한다.
'toString' in person; // true
```

`toString` 메서드가 for...in 문의 순회 대상이 되지 않는 이유는 열거할 수 없도록 정의되어 있는 프로퍼티이기 때문이다. 즉, `Object.prototype.toString` 프로퍼티의 프로퍼티 어트리뷰트 `[[Enumerable]]`의 값이 `false`이기 때문이다.

> [!NOTE]
> for...in 문은 객체의 프로토타입 체인 상에 존재하는 모든 프로토타입의 프로퍼티 중에서 프로퍼티 어트리뷰트 `[[Enumerable]]`의 값이 `true`인 프로퍼티를 순회하며 열거(enumeration)한다.

```javascript
const person = {
  name: 'Lee',
  address: 'Seoul',
  __proto__: { age: 20 }
};

for (const key in person) {
  console.log(key + ': ' + person[key]);
}

// name: Lee
// address: Seoul
// age: 20

// 프로퍼티 키가 심벌인 프로퍼티는 열거하지 않는다.
const sym = Symbol();
const obj = {
  a: 1,
  [sym]: 10
};

for (const key in obj) {
  console.log(key + ': ' + obj[key]);
}
// a : 1
```

상속받은 프로퍼티는 제외하고 객체 자신의 프로퍼티만을 열거하려면 `Object.prototype.hasOwnProperty` 메서드를 사용하여 객체 자신의 프로퍼티인지 확인해야 한다.

```javascript
const person = {
  name: 'Lee',
  address: 'Seoul',
  __proto__: { age: 20 }
};

for (const key in person) {
  if (!person.hasOwnProperty(key)) continue;
  console.log(key + ': ' + person[key]);
}
// name: Lee
// address: Seoul
```

for...in 문은 프로퍼티 열거 순서를 보장하지 않지만, 대부분의 모던 브라우저는 순서를 보장하고 숫자(프로퍼티 키 -> 문자열)인 프로퍼티 키에 대해서는 정렬을 실시한다.

```javascript
const obj = {
  2: 2,
  3: 3,
  1: 1,
  b: 'b',
  a: 'a'
};

for (const key in obj) {
  if (!obj.hasOwnProperty(key)) continue;
  console.log(key + ': ' + obj[key]);
}
/*
1: 1
2: 2
3: 3
b: b
a: a
*/
```

```javascript
const arr = [1, 2, 3];
arr.x = 10; // 배열도 객체이므로 프로퍼티를 가질 수 있다

for (const i in arr) {
  console.log(arr[i]); // 1 2 3 10
}

// arr.length는 3이다.
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]); // 1 2 3
}

// forEach 메서드는 요소가 아닌 프로퍼티는 제외한다.
arr.forEach(v => console.log(v)); // 1 2 3

// for...of는 변수 선언문에서 선언한 변수에 키가 아닌 값을 할당한다.
for (const value of arr) {
  console.log(value); // 1 2 3
}
```

### Object.keys/values/entries 메서드

for...in 문은 객체 자신의 고유 프로퍼티 뿐만 아니라 상속받은 프로퍼티도 열거한다. 따라서 `Object.prototype.hasOwnProperty` 메서드를 사용하여 객체 자신의 프로퍼티인지 확인하는 추가적인 처리가 필요하다.

따라서 객체 자신의 고유 프로퍼티만을 열거하기 위해서는 Object.keys/values/entries 메서드 사용이 권장된다.

먼저 Object.keys 메서드는 객체 자신의 열거 가능한 프로퍼티 키를 배열로 반환한다.

```javascript
const person = {
  name: 'Lee',
  address: 'Seoul',
  __proto__: { age: 20 }
};
Object.keys(person); // ['name', 'address']
```

ES8에서 도입된 Object.values 메서드는 객체 자신의 열거 가능한 프로퍼티 값을 배열로 반환한다.

```javascript
Object.values(person); // ['Lee', 'Seoul']
```

ES8에서 도입된 Object.entries 메서드는 객체 자신의 열거 가능한 프로퍼티 키와 값의 쌍의 배열을 배열에 담아 반환한다.

```javascript
Object.entries(person); // [['name', 'Lee'], ['address', 'Seoul']]
Object.entries(person).forEach(([key, value]) => console.log(key, value));
/*
name lee
address Seoul
*/
```
