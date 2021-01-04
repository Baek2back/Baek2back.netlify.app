# Web Components Lifecycle API

## Attribute changed callback

`attributeChangedCallback`은 어트리뷰트에 변경이 발생했을 때 호출된다.

```javascript
attributeChangeCallback(attrName, oldVal, newVal);
```

!> 어트리뷰트 값에 변경이 없을 때는 호출되지 않으므로 `oldVal`과 `newVal`이 같은지는 비교할 필요가 없다.

## Observed attributes

Web Components API v0에서는 기본적으로 모든 어트리뷰트에 대해 변경이 발생하면 `attributeChangedCallback`이 호출되었다. HTML 태그가 가질 수 있는 어트리뷰트 개수를 생각해보면 굉장히 낭비라는 것을 알 수 있을 것이다. 따라서 v1부터는 정확하게 변화를 감지할 어트리뷰트들 만을 명시할 수 있는 수단을 제공해준다.

```javascript
static get observedAttributes() {
  return ['name', 'value'];
}
```

## connectedCallback

```html
<script>
  class MyCustomTag extends HTMLElement {
    connectedCallback() {
      alert('hi from MyCustomTag');
      this.innerHTML =
        '<h2>' + this.getAttribute('title') + '</h2><button>click me</button>';
    }
  }

  if (!customElements.get('my-custom-tag')) {
    customElements.define('my-custom-tag', MyCustomTag);
  }
</script>
<body>
  <my-custom-tag title="Another title"></my-custom-tag>
</body>
```

위의 코드를 실행시키면 브라우저는 `alert()`를 실행시킬 것이다. 그렇다면 `connectedCallback`은 언제 호출되는 것일까? 확인을 위해 `<body>` 태그 내에서 `<my-custom-tag title="Another title"></my-custom-tag>`를 제거해보자.

기존과는 다르게 빈 페이지, 그 전에 앞서 어떠한 alert도 마주하지 못하게 된다.

```html
<script>
  class MyCustomTag extends HTMLElement {
    constructor() {
      super();
      alert('hi from MyCustomTags constructor');
    }
    connectedCallback() {
      alert('hi from MyCustomTag connected callback');
      this.innerHTML =
        '<h2>' + this.getAttribute('title') + '</h2><button>click me</button>';
    }
  }

  if (!customElements.get('my-custom-tag')) {
    customElements.define('my-custom-tag', MyCustomTag);
  }
</script>
<body></body>
```

그렇다면 `constructor`는 호출되는 지 확인하기 위해 이번에는 `connectedCallback`과 `constructor`에 모두 `alert` 메서드를 호출하는 문을 삽입한 후 결과를 확인해보자. 역시나 아무런 alert도 발생하지 않는다.

```javascript
x = document.createElement('my-custom-tag');
```

빈 페이지가 로드된 이후 개발자 도구 콘솔에서 위의 코드를 입력해보면, `constructor`에 위치시켜 놓은 `alert`이 호출되는 것을 확인할 수 있다.

```javascript
document.body.appendChild(x);
```

그 다음 위의 코드를 콘솔에 입력하면 `connectedCallback`에 위치시켜 놓은 `alert`이 호출되는 것을 확인할 수 있다.

?> `constructor`는 커스텀 엘리먼트가 생성될 때 호출되고, `connectedCallback`은 생성된 엘리먼트가 **실제로** DOM에 추가될 때 호출된다.

그렇다면 `connectedCallback`은 아무 요소의 자식으로 추가되기만 하면 호출되는 것일까?

```javascript
myEl = document.createElement('my-custom-tag');
// constructor는 호출된다.
myContainer = document.createElement('fragment');
myContainer.appendChild(myEl);
// connectedCallback은 호출되지 않는다.
```

물론 아니다. 커스텀 엘리먼트를 아무 요소의 자식으로 추가만 하는 것은 아직 실제 DOM에 반영된 것이 아니므로 `connectedCallback`을 호출시키지 못한다. 그저 `myContainer`라는 요소에 커스텀 엘리먼트를 고립시켜 둔 것 밖에 되지 않기 때문이다.

```javascript
document.body.appendChild(myContainer);
// connectedCallback이 호출된다.
```

그렇다면 간접적으로 DOM에 커스텀 엘리먼트를 추가시키는 경우는 어떨까? 위의 경우처럼 직접 커스텀 엘리먼트를 페이지에 추가하는 방법 대신 일종의 컨테이너 요소를 활용해서 간접적으로 추가하는 방식을 사용하였을 때, 컨테이너 요소에 커스텀 엘리먼트가 추가될 때에는 `connectedCallback`이 호출되지 않다가, 컨테이너 요소가 DOM에 추가될 때 호출되는 것을 알 수 있다.

```javascript
document.body.removeChild(myContainer);
document.body.appendChild(myContainer);
// connectedCallback이 호출된다.
```

거기에 커스텀 엘리먼트 자체 혹은 커스텀 엘리먼트를 포함한 요소를 DOM에서 지웠다가 다시 추가하는 경우에도 `connectedCallback`이 호출된다.

!> 즉, `connectedCallback`이 호출되었다는 것은 어떠한 방식으로든 DOM의 구성 요소가 되었다는 의미이다.

### constructor vs connectedCallback

그렇다면 `constructor`는 인스턴스 생성 시 한번만 호출되고, `connectedCallback`은 DOM에 추가될 때마다 호출되므로, 모든 로직을 `constructor`에 모아놓고, `connectedCallback`은 비워두는 것이 합리적일까? 안타깝게도 그렇지는 않다.

```html
<script>
  class MyCustomTag extends HTMLElement {
    constructor() {
      super();
      this.innerHTML =
        '<h2>' + this.getAttribute('title') + '</h2><button>click me</button>';
    }
    connectedCallback() {}
  }

  if (!customElements.get('my-custom-tag')) {
    customElements.define('my-custom-tag', MyCustomTag);
  }
</script>
```

`connectedCallback`에 위치하고 있던 `innerHTML` 코드를 `constructor`로 옮긴 뒤 `createElement`를 이용하여 커스텀 엘리먼트를 생성해보자.

```javascript
document.createElement('my-custom-tag');

Uncaught DOMException: Failed to construct 'CustomElement': The result must not have children
```

브라우저는 커스텀 엘리먼트가 최초로 생성될 때(인스턴스 생성) 자식 요소를 갖는 것을 허용하지 않는다. 더욱이 `title` 어트리뷰트의 경우에는 `constructor` 호출 시점에는 설정조차 되어 있지 않은 상태이므로 참조도 불가능하다.

```html
<script>
  class MyCustomTag extends HTMLElement {
    constructor() {
      super();
      console.log('constructor: ', this.getAttribute('title'));
    }
    connectedCallback() {
      console.log('connectedCallback: ', this.getAttribute('title'));
    }
  }
</script>
<body>
  <my-custom-tag title="Another title"></my-custom-tag>
</body>
```

```shell
constructor: null
connectedCallback: Another title
```

?> 정리하면 `connectedCallback`은 커스텀 엘리먼트를 렌더링하기 위한 모든 로직을 포함해야 한다. 그리고 렌더링 된 화면(요소)를 기준으로 이벤트를 추가하는 등의 작업을 수행해야 한다.

!> 하지만 컴포넌트의 특성에 따라 `constructor`에서 작업을 처리해야 하는 경우가 존재할 수 있다. 커스텀 엘리먼트가 초기화된 이후 실제 DOM에 반영되기 이전에 필요한 작업같은 경우이다. 네트워크 요청이 필요하다거나 화면에 렌더링될 요소와 무관한 작업들의 경우에 해당한다. 또한 해당 컴포넌트가 화면에 렌더링될 때 필요한 모든 정보를 이미 보유하고 있는 경우에도 `constructor`를 활용할 수 있다.

```javascript
class MyCustomTag extends HTMLElement {
  constructor() {
    super();
    /**
     * 가상의 리스트를 만들기 위해 데이터를 가져올 URL
     */
    this.serviceURL = 'http://company.com/services.json';

    /**
     * 내부에서 무엇인가를 추적하기 위한 카운터
     */
    this.counter = 0;

    /**
     * 마지막으로 출력된 에러 메시지
     */
    this.error;
  }
  connectedCallback() { ... }
}
```

이처럼 `constructor`의 가장 좋은 사용법은 프로퍼티 선언 용도이다. 클래스의 최상단에서 사용할 프로퍼티를 명시하면 가독성을 높일 수 있다.

?> 하지만 최근 Chrome에서는 `public`, `private` 클래스 필드를 사용할 수 있으므로, 클래스 자체에 프로퍼티를 명시할 수 있다. 가능하다면 해당 문법을 사용하도록 하자.

!> Shadow DOM을 사용하게 되면 기존 DOM과 분리(격리)된 DOM을 생성할 수 있다. Shadow DOM에서는 `constructor`에서도 어트리뷰트에 대한 참조가 가능하다. 따라서 최근의 모던 웹 컴포넌트에서는 `constructor`가 모든 기능을 수행하고, `connectedCallback`은 많이 사용되지 않는 경향을 보인다.

만약 Shadow DOM을 지원하지 않는 브라우저에 대해 코드를 작성해야 한다면 `constructor`와 `connectedCallback`의 차이를 인지하고 사용하도록 하자.

## Remaining Lifecycle API

### Disconnected callback

`disconnectedCallback`은 DOM에서 컴포넌트가 제거될 때 호출된다. 물론 특정 컴포넌트가 DOM에서 절대 제거되지 않을 것을 무조건적으로 유지할 수 있다면 정리 작업을 수행하지 않아도 상관없겠지만, 현실에서 프로젝트의 범위는 언제든지 변경될 수 있고 절대 제거되지 않을 것이라 생각했던 컴포넌트 역시 제거해야 되는 상황이 발생할 수 도 있다.

만약 업데이트 된 데이터를 얻기 위해 30초 마다 서버에 쿼리를 날린다고 가정해보자. 그 상황에서 `removeChild(element)`를 이용하여 부모 컨테이너에서 제거하더라도 여전히 타이머는 실행되고 서버에 쿼리를 날리게 될 것이다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Cleanup Component</title>
    <script>
      class CleanupComponent extends HTMLElement {
        connectedCallback() {
          this.counter = 100;
          setInterval(() => this.update(), 1000);
        }
        update() {
          this.innerHTML = this.counter;
          this.counter--;
          console.log(this.counter);
        }
      }
      customElements.define('cleanup-component', CleanupComponent);
    </script>
  </head>
  <body>
    <cleanup-component></cleanup-component>
    <button
      onclick="document.body.removeChild(document.querySelector('cleanup-component'))"
    >
      remove
    </button>
  </body>
</html>
```

버튼을 클릭하면 컴포넌트는 DOM에서 제거되어 더이상 화면에 보이지 않지만 콘솔에는 지속적으로 `counter`의 값이 출력되게 된다. 만약 네트워크 요청을 보내는 상황이라면 불필요한 요청을 계속 보내게 되어 리소스를 낭비하게 될 것이다.

따라서 이러한 상황에서 우리는 `disconnectedCallback`을 꺼내들고자 한다. 컴포넌트가 DOM에서 제거될 때 모든 이벤트 리스너를 정리하고, 타이머를 제거시키게끔 하는 것이다.

```javascript
class CleanupComponent extends HTMLElement {
  connectedCallback() {
    this.counter = 100;
    this.timer = setInterval(() => this.update(), 1000);
  }
  update() {
    this.innerHTML = this.counter;
    this.counter--;
    console.log(this.counter);
  }
  disconnectedCallback() {
    clearInterval(this.timer);
  }
}
customElements.define('cleanup-component', CleanupComponent);
```

콘솔 창을 확인해보면 컴포넌트가 DOM에서 제거된 이후에는 출력되지 않는 것을 확인할 수 있다.

### Adopted callback

`adoptedCallback`은 커스텀 엘리먼트가 다른 `document`로 이동했을 때 호출된다.

일반적으로 HTML은 페이지 당 하나의 `document`를 갖게 된다. 예외의 경우는 `<iframe>`을 사용할 때이다. 그러나 이 메서드가 호출되는 경우는 굉장히 드물기 때문에 일단 존재한다는 사실만 알아두도록 하자.

## Reference

- [Web Components in Action](https://www.amazon.com/Web-Components-Action-Ben-Farrell/dp/1617295779)
