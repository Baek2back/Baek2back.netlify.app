# Chapter 01. 리팩터링: 첫 번째 예시

## The Starting Program

다양한 연극을 외주로 받아 공연하는 극단이 있다고 가정해보자. 공연 요청이 들어오면 장르와 관객 수를 기준으로 비용을 책정한다. 현재 이 극단은 비극(tragedy)와 희극(comedy)만 공연하며, 공연료와 별개로 포인트(volume credit)를 지급해 다음 의뢰 시 공연료 할인 기능도 제공한다.

이때 극단에서 공연할 연극에 대한 정보를 간단한 JSON 파일로 만들었다고 생각해보자.

```json
// plays.json
{
  "hamlet": { "name": "Hamlet", "type": "tragedy" },
  "as­like": { "name": "As You Like It", "type": "comedy" },
  "othello": { "name": "Othello", "type": "tragedy" }
}
```

공연료 청구서에 들어갈 비용에 대한 정보 역시 JSON 파일로 구성되어 있다고 생각해보자.

```json
// invoices.json
{
  "customer": "BigCo",
  "performances": [
    {
      "playID": "hamlet",
      "audience": 55
    },
    {
      "playID": "as-like",
      "audience": 35
    },
    {
      "playID": "othello",
      "audience": 40
    }
  ]
}
```

그리고 청구서를 출력하는 간단한 함수가 있다 생각해보자.

```javascript
function statement(invoice, plays) {
  let totalAmout = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format;

  for (let perf of invoice.performances) {
    const play = plays[perf.playId];
    let thisAmount = 0;

    switch (play.type) {
      case 'tragedy': // 비극
        thisAmount = 40000;
        if (perf.audience > 30) {
          thisAmount += 1000 * (perf.audience - 30);
        }
        break;
      case 'comedy':
        this.Amount = 30000;
        if (perf.audience > 20) {
          thisAmount += 10000 + 500 * (perf.audience - 20);
        }
        thisAmount += 300 * perf.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
    }
    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ('comedy' === play.type) volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    result += `  ${play.name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

```
청구 내역 (고객명: BigCo)
  Hamlet: $650.00 (55석)
  As You Like It: $580.00 (35석)
  Othello: $500.00 (40석)
총액: $1,730.00
적립 포인트: 47점
```

## 소감

우선 프로그램은 잘 동작하고, 컴파일러는 코드가 지저분하든 깔끔하든 전혀 개의치 않으므로 당장의 문제는 없어보인다. 하지만 코드를 수정하기 위해서는 필연적으로 사람이 투입되고, 그때 설계가 나쁜 시스템은 수정하기 굉장히 어렵다. 수정하기 어렵다는 의미는 실수가 잦고 그로 인해 버그를 양산할 가능성이 높아진다는 의미이다.

?> 프로그램이 새로운 기능을 추가하기에 편한 구조가 아닌 경우, 먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다.

이제 몇 가지 추가 기능이 필요해진 상황이라 생각해보자.

- **청구 내역을 HTML로 출력하는 기능**

  - 청구 결과에 문자열을 추가하는 문장 각각을 조건문으로 감싸야 하므로 `statement()` 함수의 복잡도 향상에 크게 기여할 것이다.

- **배우들이 더 많은 장르의 연극을 공연하고 싶어하는 경우**

  - 공연료 산정과 적립 포인트 계산법에 영향을 줄 것이다.

물론 기존의 코드가 잘 작동하고 추후에 변경할 일이 절대 존재하지 않는다면 상관없었겠지만 요구 사항이 변동되고 다른 사람이 읽고 이해해야 할 일이 생겼을 때 로직을 한눈에 파악하기 어렵다면 그에 따른 대책을 마련해야 한다.

## statement() 함수 쪼개기

먼저 긴 함수를 리팩터링할 때는 먼저 전체 동작을 각각의 부분으로 나눌 수 있는 지점을 찾는다. 그렇다면 먼저 `switch` 문이 눈에 띌 것이다.

```javascript
...
for (let perf of invoice.performances) {
  const play = plays[perf.playID];
  let thisAmount = 0;
  // (switch 문)
  switch (play.type) {
    case 'tragedy':
      thisAmount = 40000;
      if (perf.audience > 30) {
        thisAmount += 1000 * (perf.audience - 30);
      }
      break;
    case 'comedy':
      thisAmount = 30000;
      if (perf.audience > 20) {
        thisAmount += 10000 + 500 * (perf.audience - 20);
      }
      thisAmount += 300 * perf.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르: ${play.type}`);
  }
  // 포인트를 적립한다.
  volumeCredits += Math.max(perf.audience - 30, 0);
  // 희극 관객 5명마다 추가 포인트를 제공한다.
  if ('comedy' === play.type) volumeCredits += Math.floor(perf.audience / 5);
  // 청구 내역을 출력한다.
  result += `  ${play.name}: ${format(thisAmount / 100)} (${
    perf.audience
  }석)\n`;
  totalAmount += thisAmount;
}
...
```

먼저 `switch` 문을 살펴보면 한 번의 공연에 대한 요금을 계산하고 있지만, 이러한 사실은 코드를 일일히 분석해서 얻은 정보이다. 이런 식으로 파악한 정보는 휘발성이 높기 때문에 재빨리 코드 조각을 별도의 함수로 추출해 두는 것이 좋다. 그때 추출한 함수에는 해당 코드가 하는 일을 설명하는 이름을 지어주어야 한다. 여기서는 `amountFor` 정도가 적당해 보인다.

먼저 별도의 함수로 추출했을 때는 유효범위를 벗어나는 변수, 즉 새로운 함수에서는 곧바로 사용할 수 없는 변수가 있는지 확인한다. 위의 코드에서는 `perf`,`play`,`thisAmount`가 여기에 속한다. 이때 `perf`와 `play`는 새로운 함수에서 필요로 하지만 내부에서 값을 변경하지 않기 때문에 매개변수로 전달하면 될 것이다. 그러나 `thisAmount`의 경우 함수 내부에서 값이 바뀌기 때문에 조심해서 다뤄야 한다.

```javascript
function statement(invoice, plays) {
  ...
  function amountFor(perf, play) {
    // (값이 바뀌지 않는 변수는 매개변수로 전달)
    let thisAmount = 0; // (변수 초기화)

    switch (play.type) {
      case 'tragedy':
        thisAmount = 40000;
        if (perf.audience > 30) {
          thisAmount += 1000 * (perf.audience - 30);
        }
        break;
      case 'comedy':
        thisAmount = 30000;
        if (perf.audience > 20) {
          thisAmount += 10000 + 500 * (perf.audience - 20);
        }
        thisAmount += 300 * perf.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
    }

    return thisAmount; // (함수 내부에서 값이 바뀌는 변수 반환)
  }
}
```

이제 `statement()`에서는 `thisAmount` 값을 채울 때 방금 추출한 `amountFor()` 함수를 호출하게끔 하자.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format;
  for (let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = amountFor(perf, play); // (추출한 함수 이용)
    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (play.type === 'comedy') volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    result += `  ${play.name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function amountFor(perf, play) { ... }
}
```

아무리 간단한 수정이라도 리팩터링 후에는 항상 테스트하는 습관을 들여야 한다. 한 가지를 수정할 때마다 테스트를 수행하면 오류가 발생하더라도 변경 폭이 작기 때문에 살펴볼 범위도 좁아서 문제를 찾고 해결하기 훨씬 수월하다. 한 번에 너무 많이 수정하려다 실수를 저지르면 디버깅이 어려워지므로 결과적으로는 작업 시간이 늘어나게 된다.

?> 리팩터링은 프로그램 수정을 작은 단계로 나누어 진행한다. 그래서 중간에 실수하더라도 버그를 쉽게 찾을 수 있다.

지금 예시는 자바스크립트로 작성되었으므로 중첩 함수로 만들 수 있었다. 이러한 경우 외부 함수에서 사용하던 변수를 새로 추출한 함수에 매개변수로 전달할 필요가 없어서 편해진다.

!> 다만 여기에서는 `perf`와 `play` 모두 `for...of` 문 안에서 선언된 변수이므로 중첩 함수가 접근하는 것은 불가능하므로 어차피 매개변수로 넘겨야 한다.

함수를 추출하고 나면 추출된 함수 코드를 자세히 들여다보면서 지금보다 명확하게 표현할 수 있는 간단한 방법은 없는지 검토한다. 가장 먼저 변수의 이름을 더 명확하게 바꿔보자. 가령 `thisAmount`를 `result`로 변경할 수 있을 것이다. 또한 첫 번째 인수인 `perf`를 보다 명확하게 `aPerformance`로 리팩터링 해보자.

```javascript
function amountFor(aPerformance, play) {
  // perf → aPerformance (보다 명확한 이름으로 변경)
  let result = 0; // thisAmount → result (보다 명확한 이름으로 변경)
  switch (play.type) {
    case 'tragedy':
      result = 40000;
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case 'comedy':
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르: ${play.type}`);
  }
  return result;
}
```

자바스크립트와 같은 동적 타입 언어를 사용할 때는 타입이 드러나게 작성하면 도움이 된다. 따라서 저자의 경우 매개변수 이름에 접두어로 타입 이름을 적는데, 지금처럼 매개변수의 역할이 뚜렷하지 않은 경우 부정 관사(a/an)을 붙인다.

?> 컴퓨터가 이해하는 코드는 바보도 작성할 수 있다. 사람이 이해도록 작성하는 프로그래머가 진정한 실력자다.

그렇다면 이렇게 이름을 바꿀만한 가치가 있는 것일까? 좋은 코드라면 하는 일이 명확히 드러나야 하므로 명확성을 높이기 위해 이름 바꾸기는 주저할 필요가 없다. 그렇다면 이제 다음으로 `play` 매개변수의 이름을 바꿀 차례인데, 이 변수는 조금 다르게 처리해주어야 한다.

> **play 변수 제거하기**

`amountFor()`의 매개변수 중에서 `aPerformance`는 `for...of` 문의 변수로부터 제공되기 때문에 매 루프마다 자연스레 값이 변경된다. 그러나 `play`는 개별 공연(`aPerformance`)에서 얻기 때문에 애초에 매개변수로 전달할 필요가 없다. 그냥 `amountFor()` 안에서 다시 계산하면 된다. 이처럼 긴 함수를 잘게 쪼갤 때에는 `play`같은 변수를 최대한 제거한다. 이런 임시 변수들 때문에 로컬 범위에 존재하는 이름이 늘어나서 추출 작업이 복잡해지기 때문이다.

먼저 대입문(`=`)의 우변을 함수로 추출하자.

```javascript
function playFor(aPerformance) {
  return plays[aPerformance.playID];
}

function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format;
  for (let perf of invoice.performances) {
    // const play = plays[perf.playID]; (① 우변을 함수로 추출)
    // const play = playFor(perf); (② 인라인 된 변수는 제거)
    let thisAmount = amountFor(perf, playFor(perf)); // (변수 인라인)
    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (playFor(perf).type === 'comedy')
      // (변수 인라인)
      volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    // (변수 인라인)
    result += `  ${playFor(perf).name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function playFor(aPerformance) {}
  function amountFor(perf, play) {}
}
```

이제 변수를 인라인한 덕분에 `amountFor()`에서 `play` 매개변수를 제거할 수 있게 되었다. 이 작업은 두 단계로 진행되는 데, 먼저 새로 만든 `playFor()`를 사용하도록 `amountFor()`를 수정하고 `play` 매개변수를 삭제한다.

```javascript
function amountFor(aPerformance) {
  // (play 매개변수 삭제)
  let result = 0;
  switch (
    playFor(aPerformance).type // (play를 playFor() 호출로 변경)
  ) {
    case 'tragedy':
      result = 40000;
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case 'comedy':
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르: ${playFor(aPerformance).type}`); // (play를 playFor() 호출로 변경)
  }
  return result;
}

function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format;
  for (let perf of invoice.performances) {
    let thisAmount = amountFor(perf); // (필요 없어진 매개변수 제거)
    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (playFor(perf).type === 'comedy')
      // (변수 인라인)
      volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    // (변수 인라인)
    result += `  ${playFor(perf).name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function playFor(aPerformance) {}
  function amountFor(perf) {} // (play 매개변수 삭제)
}
```

이전 코드는 루프를 한 번 돌 때마다 공연을 조회한 것에 반해 리팩터링한 코드에서는 세 번이나 조회하게 된다. 추후에 성능과 리팩터링과의 관계는 설명하겠지만, 우선은 이렇게 변경해도 성능에 큰 영향은 없다. 설사 심각하게 느려진다 하더라도 제대로 리팩터링 된 코드 베이스는 그렇지 않은 코드보다 성능을 개선하기가 훨씬 수월하다.

먼저 지역 변수를 제거해서 얻는 가장 큰 장점은 추출 작업이 훨씬 쉬워진다는 것이다. 유효 범위를 신경 써야 할 대상이 줄어들기 때문이다.

이제 `amountFor()`에 전달할 인수를 모두 처리했으니, 이제 이 함수를 호출하는 코드로 돌아가보면 `amountFor()`의 경우 임시 변수인 `thisAmount`에 값을 설정하는 데 사용되는데, 그 값이 다시 바뀌지는 않기 때문에 변수를 인라인해보자.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format;
  for (let perf of invoice.performances) {
    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (playFor(perf).type === 'comedy')
      volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    // (thisAmount 변수를 인라인)
    result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    // (thisAmount 변수를 인라인)
    totalAmount += amountFor(perf);
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function playFor(aPerformance) {}
  function amountFor(perf) {} // (play 매개변수 삭제)
}
```

> **적립 포인트 계산 코드 추출하기**

앞서 `play` 변수를 제거한 결과 지역 변수가 하나 제거되어 적립 포인트 계산 부분을 추출하기가 훨씬 쉬워졌다.

물론 아직 처리해야 할 변수가 아직 두 개 더 남아 있다. 여기서도 `perf`는 간단히 전달만 하면 되지만, `volumeCredits`는 반복문을 돌 때마다 값을 누적해야 하기 때문에 까다롭다. 이 상황에서 최선의 방법은 추출한 함수에서 `volumeCredits`의 복제본을 초기화한 뒤 계산 결과를 반환토록 하는 것이다.

```javascript
function volumeCreditsFor(aPerformance) {
  let volumeCredits = 0;
  volumeCredits += Math.max(aPerformance.audience - 30, 0);
  if (playFor(aPerformance).type === 'comedy')
    volumeCredits += Math.max(aPerformance.audience / 5);
  return volumeCredits;
}

function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-us', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf); // (추출한 함수를 이용해 값을 누적)

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

> **format 변수 제거하기**

임시 변수는 자신이 속한 루틴에서만 의미가 있기 때문에 루틴이 길고 복잡해지기 쉽다. 다음으로 진행할 리팩터링은 이러한 변수들을 제거하는 것이다. 그 중에서도 가장 먼저 눈에 띄는 것은 `format`인데 이는 함수 객체를 가리키고 있는 상태이다. 따라서 이를 함수를 직접 선언해 사용하는 형태로 바꾸어 보자.

```javascript
function format(aNumber) {
  return new Intl.NumberFormat('en-us', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format(aNumber);
}

function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf); // (추출한 함수를 이용해 값을 누적)

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function format(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

하지만 `format`이라는 함수 이름은 이 함수가 하는 일을 충분히 설명해 주지 못한다는 느낌이 든다. 템플릿 문자열 안에서 사용될 이름이기 때문에 `formatAsUSD`라고 하기에는 너무 장황하다. 이 함수의 본질은 화폐 단위 맞추기이기 때문에 그러한 느낌을 살릴 수 있는 이름으로 바꾸어보자.

```javascript
// (함수 이름 변경)
function usd(aNumber) {
  return new Intl.NumberFormat('en-us', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format(aNumber / 100);
  // (단위 변환 로직도 이 함수 안으로 이동)
}

function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf); // (추출한 함수를 이용해 값을 누적)

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function usd(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

앞서 이름을 변경할 때 여러 차례 등장하는 100으로 나누는 코드도 추출한 함수로 옮긴 상태이다.

> **volumeCredits 변수 제거하기**

다음 차례는 `volumeCredits`다. 이 변수는 매 루프마다 값을 누적하기 때문에 이 누적되는 부분을 따로 빼내고, `volumeCredits` 변수를 선언하는 문장을 반복문 바로 위로 옮겨보자.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }
  // (변수 선언을 반복문 바로 위로 이동)
  let volumeCredits = 0;
  // (값을 누적하는 로직을 별도의 for 문으로 분리)
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function usd(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

그 다음 `volumeCredits` 값을 계산하는 코드를 함수로 추출해보자.

```javascript
function totalVolumeCredits() {
  let volumeCredits = 0;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }
  return volumeCredits;
}

function statement(invoice, plays) {
  let totalAmount = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }
  // (값 계산 로직을 함수로 추출)
  let volumeCredits = totalVolumeCredits();
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;

  function totalVolumeCredits() {}
  function usd(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

함수로 추출한 이후에는 `volumeCredits` 변수를 인라인 해보자.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${usd(totalAmount)}\n`;
  // (변수 인라인)
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function totalVolumeCredits() {}
  function usd(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

반복문을 쪼개는 작업에 있어서 성능 저하를 우려할 수도 있지만, 이 정도 규모의 중복은 성능에 미치는 영향이 미미한 경우가 많다. 때로는 리팩터링이 성능에 상당한 영향을 주기도 하지만, 잘 다듬어진 코드일 수록 성능 개선 작업이 수월해 진다는 점을 잊지말자. 리팩터링 과정에서 성능이 많이 저하되었다면 리팩터링 이후에 성능 개선을 시도해보자. 대체로 더 깔끔하면서 빠르게 동작하는 코드를 얻을 수 있을 것이다.

이제 `totalAmount`도 앞의 `volumeCredits`와 동일하게 제거해보자. 먼저 반복문을 쪼개고, 변수 초기화 문을 옮긴 다음, 함수를 추출한다. 그 다음 변수를 인라인하는 과정까지 진행해보자.

```javascript
function totalAmount() {
  let result = 0;
  for (let perf of invoice.performances) {
    result += amountFor(perf);
  }
  return result;
}

function statement(invoice, plays) {
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {}
  function totalVolumeCredits() {}
  function usd(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

## 중간 점검: 난무하는 중첩 함수

그렇다면 현재까지의 리팩터링 결과를 한 번 살펴보자.

```javascript
function statement(invoice, plays) {
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  // 중첩 함수들
  function totalAmount() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += amountFor(perf);
    }
    return result;
  }
  function totalVolumeCredits() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += volumeCreditsFor(perf);
    }
    return result;
  }
  function usd(aNumber) {
    return new Intl.NumberFormat('en-us', {
      style: 'currency',
      currency: 'USD',
      minimumFractionDigits: 2
    }).format(aNumber / 100);
  }
  function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(aPerformance.audience - 30, 0);
    if (playFor(aPerformance).type === 'comedy')
      result += Math.floor(aPerformance.audience / 5);
    return result;
  }
  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }
  function amountFor(aPerformance) {
    let result = 0;
    switch (playFor(aPerformance).type) {
      case 'tragedy':
        result = 40000;
        if (aPerformance.audience > 30) {
          result += 1000 * (aPerformance.audience - 30);
        }
        break;
      case 'comedy':
        result = 30000;
        if (aPerformance.audience > 20) {
          result += 10000 * 500 * (aPerformance.audience - 20);
        }
        result += 300 * aPerformance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르: ${playFor(aPerformance).type}`);
    }
    return result;
  }
}
```

코드 구조가 한결 나아진 것을 확인할 수 있다 `statement()` 함수가 실행하는 문은 급격히 감소하였으며 출력할 문장을 생성하는 일만 하게 된다. 계산 로직은 보조 함수로 추출하였고, 결과적으로 각 계산 과정은 물론 전체 흐름을 이해하기 쉬워졌다.

## 계산 단계와 포맷팅 단계 분리하기

이제 실질적인 기능 변경, `statement()`의 HTML 버전을 만들어보자. 계산 코드가 모두 분리되었기 때문에 최상단 코드에 대응하는 HTML 버전만 작성하면 될 것이다. 하지만 분리된 계산 함수들이 텍스트 버전인 `statement()` 내부의 중첩 함수로 존재하기 때문에 이 모두를 그대로 복사하는 방식으로 HTML 버전을 만드는 것은 비효율적이다. 텍스트 버전, HTML 버전 함수 모두가 똑같은 계산 함수를 사용하게끔 만들어 보자.

먼저 `statement()`에 필요한 데이터를 처리하고, 다음 단계에서는 앞서 처리한 결과를 텍스트 혹은 HTML로 표현하도록 만들어보자. 정리하면 첫 번째 단계에서는 두 번째 단계로 전달할 중간 데이터 구조를 생성하는 것이다.

```javascript
function statement(invoice, plays) {
  // (본문 전체를 별도 함수로 추출)
  return renderPlainText(invoice, plays);
}

function renderPlainText(invoice, plays) {
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {}
  function totalVolumeCredits() {}
  function usd(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

이제 다음으로 두 단계 사이의 중간 데이터 구조 역할을 할 객체를 만들어서 `renderPlainText()`에 인수로 전달한다.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  // (중간 데이터 구조를 인수로 전달)
  return renderPlainText(statementData, invoice, plays);
}

// (중간 데이터 구조를 인수로 전달)
function renderPlainText(data, invoice, plays) {
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {}
  function totalVolumeCredits() {}
  function usd(aNumber) {}
  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}
```

이번에는 `renderPlainText()`의 다른 두 인수(`invoice`,`plays`)를 살펴보자. 이 인수들을 통해 전달되는 데이터를 모두 방금 만든 중간 데이터 구조로 옮기면, 계산 관련 코드는 전부 `statement()` 함수로 모으고 `renderPlainText()`는 `data` 매개변수로 전달된 데이터만 처리하게 만들 수 있다.

가장 먼저 고객 정보부터 중간 데이터 구조로 옮겨보자.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  // (고객 데이터를 중간 데이터로 옮김)
  statementData.customer = invoice.customer;
  return renderPlainText(statementData, invoice, plays);
}

function renderPlainText(data, invoice, plays) {
  // (고객 데이터를 중간 데이터로부터 얻음)
  let result = `청구 내역 (고객명: ${data.customer})\n`;
  for (let perf of invoice.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;
  ...
}
```

같은 방식으로 공연 정보까지 중간 데이터 구조로 옮기고 나면 `renderPlainText()`의 `invoice` 매개변수를 삭제해도 된다.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  // (공연 정보를 중간 데이터로 옮김)
  statementData.performances = invoice.performances;
  // (필요 없어진 인수 삭제)
  return renderPlainText(statementData, plays);
}

// (필요 없어진 인수 삭제)
function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명: ${data.customer})\n`;
  for (let perf of data.performances) {
    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {
    let result = 0;
    for (let perf of data.performances) {
      result += amountFor(perf);
    }
    return result;
  }

  function totalVolumeCredits() {
    let result = 0;
    for (let perf of data.performances) {
      result += volumeCreditsFor(perf);
    }
    return result;
  }
}
```

이제 연극 제목도 중간 데이터 구조에서 가져오도록 해보자. 이를 위해 공연 정보 레코드에 연극 데이터를 추가해야 한다.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  return renderPlainText(statementData, plays);

  function enrichPerformance(aPerformance) {
    // (얕은 복사 수행)
    const result = Object.assign({}, aPerformance);
    return result;
  }
}
```

일단은 공연 객체를 복사하기만 했지만, 추후에 새로 만든 레코드에 데이터를 채울 것이다. 이때 얕은 복사를 수행한 이유는 불변 데이터(Immutable)로 취급하기 위해서이다.

이제 연극 정보를 담을 자리가 마련되었으니 실제로 데이터를 담아보자. `playFor()` 함수를 `statement()`로 옮긴다.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  return renderPlainText(statementData, plays);

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result); // (중간 데이터에 연극 정보를 저장)
    return result;
  }

  // (renderPlainText()의 중첩 함수 였던 playFor()를 statement()로 옮김)
  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }
}
```

그런 다음 `renderPlainText()` 안에서 `playFor()`를 호출하던 부분을 중간 데이터를 사용하도록 바꾼다.

```javascript
function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명: ${data.customer})\n`;
  for (let perf of data.performances) {
    // 청구 내역을 출력한다.
    result += `  ${perf.play.name}: ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(aPerformance.audience - 30, 0);
    if (aPerformance.play.type === 'comedy')
      result += Math.floor(aPerformance.audience / 5);
    return result;
  }

  function amountFor(aPerformance) {
    let result = 0;
    switch (aPerformance.play.type) {
      case 'tragedy':
        result = 40000;
        if (aPerformance.audience > 30) {
          result += 1000 * (aPerformance.audience - 30);
        }
        break;
      case 'comedy':
        result = 30000;
        if (aPerformance.audience > 20) {
          result += 10000 + 500 * (aPerformance.audience - 20);
        }
        result += 300 * aPerformance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르: ${aPerformance.play.type}`);
    }
    return result;
  }
}
```

이어서 `amountFor()`도 비슷한 방식으로 옮겨보자.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  return renderPlainText(statementData, plays);

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    return result;
  }

  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}

function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명: ${data.customer})\n`;
  for (let perf of data.performances) {
    // 청구 내역을 출력한다.
    result += `  ${perf.play.name}: ${usd(perf.amount)} (${perf.audience}석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {
    let result = 0;
    for (let perf of data.performances) {
      result += perf.amount;
    }
    return result;
  }
}
```

다음으로 적립 포인트 계산 부분을 옮겨보자.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  return renderPlainText(statementData, plays);

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    result.volumeCredits = volumeCreditsFor(result);
    return result;
  }

  function volumeCreditsFor(aPerformance) {}
  function playFor(aPerformance) {}
  function amountFor(aPerformance) {}
}

function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명: ${data.customer})\n`;
  for (let perf of data.performances) {
    // 청구 내역을 출력한다.
    result += `  ${perf.play.name}: ${usd(perf.amount)} (${perf.audience}석)\n`;
  }
  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;

  function totalVolumeCredits() {
    let result = 0;
    for (let perf of data.performances) {
      result += perf.volumeCredits;
    }
    return result;
  }
}
```

마지막으로 총합을 구하는 부분을 옮긴다.

```javascript
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  statementData.totalAmount = totalAmount(statementData);
  statementData.totalVolumeCredits = totalVolumeCredits(statementData);
  return renderPlainText(statementData, plays);

  function totalAmount(data) {}
  function totalVolumeCredits(data) {}
}

function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명: ${data.customer})\n`;
  for (let perf of data.performances) {
    // 청구 내역을 출력한다.
    result += `  ${perf.play.name}: ${usd(perf.amount)} (${perf.audience}석)\n`;
  }
  result += `총액: ${usd(data.totalAmount)}\n`;
  result += `적립 포인트: ${data.totalVolumeCredits}점\n`;
  return result;
}
```

물론 총합을 구하는 두 함수 `totalAmount()`와 `totalVolumeCredits()`의 본문에서 `statementData` 변수를 사용할 수도 있지만 정확히 매개변수로 전달하는 방식이 더 명확해 보인다.

이제는 반복문을 파이프라인으로 바꿔보자.

```javascript
function renderPlainText(data, plays) {
  function totalAmount(data) {
    return data.performances.reduce((total, p) => total + p.amount, 0);
  }
  function totalVolumeCredits(data) {
    return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
  }
}
```

이제 첫 단계인 `statement()`에 필요한 데이터 처리에 해당하는 코드를 모두 함수로 분리해보자.

```javascript
function statement(invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays));
}

// (중간 데이터 생성을 전담한다)
function createStatementData(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  statementData.totalAmount = totalAmount(statementData);
  statementData.totalVolumeCredits = totalVolumeCredits(statementData);
  return statementData;
}
```

이제 두 단계가 명확히 분리되었으니 각 코드를 별도의 파일에 저장한다.

```javascript
/* statement.js */
import createStatementData from './createStatementData.js';

/* createStatementData.js */
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {}
  function palyFor(aPerformance) {}
  function amountFor(aPerformance) {}
  function volumeCreditsFor(aPerformance) {}
  function totalAmount(data) {}
  function totalVolumeCredits(data) {}
}
```

이제 HTML 버전을 완성시켜 보자.

```javascript
/* statement.js */
function htmlStatement(invoice, plays) {
  // (중간 데이터 생성 함수를 공유)
  return renderHtml(createStatementData(invoice, plays));
}

function renderHtml(data) {
  let result = `<h1>청구 내역 (고객명: ${data.customer})</h1>\n`;
  result += `<table>\n`;
  result += `<tr><th>연극</th><th>좌석 수</th><th>금액</th></tr>`;
  for (let perf of data.performances) {
    result += `  <tr><td>${perf.play.name}</td><td>(${perf.audience}석)</td></tr>`;
    result += `<td>${usd(perf.amount)}</td></tr>\n`;
  }
  result += `</table>\n`;
  result += `<p>총액: <em>${usd(data.totalAmount)}</em></p>\n`;
  result += `<p>적립 포인트: <em>${data.totalVolumeCredits}</em>점</p>\n`;
  return result;
}

function usd(aNumber) {}
```

?> `usd()`를 `renderHtml()`에서도 사용할 수 있도록 최상위로 옮겼다.

## 중간 점검: 두 파일(과 두 단계)로 분리됨

현재 코드는 두 개의 파일로 구성된다.

```javascript
/* statement.js */
import createStatementData from './createStatementData.js';

function statement(invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays));
}

function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명: ${data.customer})\n`;
  for (let perf of data.performances) {
    result += `  ${perf.play.name}: ${usd(perf.amount)} (${perf.audience}석\n`;
  }
  result += `총액: ${usd(data.totalAmount)}\n`;
  result += `적립 포인트: ${data.totalVolumeCredits}점\n`;
  return result;
}

function htmlStatement(invoice, plays) {
  return renderHtml(createStatementData(invoice, plays));
}

function renderHtml(data) {
  let result = `<h1>청구 내역 (고객명: ${data.customer})</h1>\n`;
  result += `<table>\n`;
  result += `<tr><th>연극</th><th>좌석 수</th><th>금액</th></tr>`;
  for (let perf of data.performances) {
    result += `  <tr><td>${perf.play.name}</td><td>(${perf.audience}석)</td></tr>`;
    result += `<td>${usd(perf.amount)}</td></tr>\n`;
  }
  result += `</table>\n`;
  result += `<p>총액: <em>${usd(data.totalAmount)}</em></p>\n`;
  result += `<p>적립 포인트: <em>${data.totalVolumeCredits}</em>점</p>\n`;
  return result;
}

function usd(aNumber) {
  return new Intl.NumberFormat('en-us', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2
  }).format(aNumber / 100);
}

/* createStatementData.js */

export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;
}

function enrichPerformance(aPerformance) {
  const result = Object.assign({}, aPerformance);
  result.play = playFor(result);
  result.amount = amountFor(result);
  result.volumeCredits = volumeCreditsFor(result);
  return result;
}

function playFor(aPerformance) {
  return plays[aPerformance.playID];
}

function amountFor(aPerformance) {
  let result = 0;
  switch (aPerformance.play.type) {
    case 'tragedy':
      result = 40000;
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case 'comedy':
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르: ${aPerformance.play.type}`);
  }
  return result;
}

function volumeCreditsFor(aPerformance) {
  let result = 0;
  result += Math.max(aPerformance.audience - 30, 0);
  if (aPerformance.play.type === 'comedy')
    result += Math.floor(aPerformance.audience / 5);
  return result;
}

function totalAmount(data) {
  return data.performances.reduce((total, p) => total + p.amount, 0);
}

function totalVolumeCredits(data) {
  return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
}
```

전체 라인 수는 늘었지만 추가된 코드 덕분에 전체 로직을 구성하는 요소 각각이 더 뚜렷해지고, 계산하는 부분과 출력 형식을 다루는 부분이 분리되었다. 이처럼 모듈화를 진행하면 각 부분이 하는 일과 그 부분들이 맞물려 돌아가는 과정을 파악하기 쉬워진다. 결국 모듈화를 한 덕분에 코드를 중복하지 않고도 HTML 버전을 만들 수 있게 되었다.

## 다형성을 활용해 계산 코드 재구성하기

이번에는 연극 장르를 추가하고 장르마다 공연료와 적립 포인트 계산법을 다르게 지정하도록 수정해보자. 이는 해당 계산을 수행하는 함수에서 조건문을 수정하는 방식으로 바꿀 수 있을 것이다. `amountFor()` 함수를 보면 연극 장르 별로 계산 방법이 다르다는 것을 알 수 있다.

여기서는 일단 객체 지향의 핵심 특성인 다형성(polymorphism)을 활용해보자. 우선 상속 계층을 구성해서 희극 서브클래스와 비극 서브클래스가 각각의 구체적인 계산 로직을 정의하게끔 하는 것이다. 호출부에서는 공연료 계산 함수를 호출하기만 하고, 희극인지 비극인지에 따라 정확한 계산 방식은 언어 차원에서 처리하도록 하는 것이다. 그러면 우선 상속 계층부터 정의해야 하는데 공연료와 적립 포인트 계산 함수를 담을 클래스를 만들어보자.

> **공연료 계산기 만들기**

핵심은 각 공연의 정보를 중간 데이터 구조에 채워주는 `enrichPerformance()` 함수이다. 현재 이 함수는 `amountFor()`와 `volumeCreditsFor()`를 호출하여 공연료와 적립 포인트를 계산한다. 이제 이 두 함수를 전용 클래스로 옮겨보자. 이 클래스는 공연 관련 데이터를 계산하는 함수들로 구성되므로 `PerformanceCalculator` 정도의 이름이 적당할 것 같다.

```javascript
/* createStatementData.js */

// (공연료 계산기 클래스)
class PerformanceCalculator {
  constructor(aPerformance) {
    this.performance = aPerformance;
  }
}

function enrichPerformance(aPerformance) {
  // (공연료 계산기 생성)
  const calculator = new PerformanceCalculator(aPerformance);
  const result = Object.assign({}, aPerformance);
  result.play = playFor(result);
  result.amount = amountFor(result);
  result.volumeCredits = volumeCreditsFor(result);
  return result;
}
```

기존 코드에서 몇 가지 동작을 이 클래스로 옮겨보자. 먼저 연극 레코드부터 시작해보자.

```javascript
class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }
}

function enrichPerformance(aPerformance) {
  // (공연 정보를 계산기로 전달)
  const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance));
  const result = Object.assign({}, aPerformance);
  result.play = calculator.play;
  result.amount = amountFor(result);
  result.volumeCredits = volumeCreditsFor(result);
  return result;
}
```

> **함수들을 계산기로 옮기기**