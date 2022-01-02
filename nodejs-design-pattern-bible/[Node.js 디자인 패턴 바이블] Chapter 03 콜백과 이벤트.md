# [Node.js 디자인 패턴 바이블] Chapter 03 콜백과 이벤트



**이번 절에서 배울 것들**

- 콜백 패턴 - 어떻게 동작하며 Node.js에서 사용되는 관례는 무엇인지 그리고 매우 흔한 위험요소를 어떻게 다룰 것인가.
- 관찰자 패턴 - Node.js에서 EventEmitter 클래스를 사용하여 구현을 어떻게 할 것인가.



**목차**

- 3-1 콜백 패턴
  - 연속 전달 방식
  - 동기? 비동기?
  - Node.js 콜백 규칙
- 3-2 관찰자 패턴(The observer pattern)
  - EventEmitter 클래스
  - EventEmitter 생성 및 사용
  - 오류 전파
  - 관찰 가능한 객체 만들기
  - EventEmitter와 메모리 누수
  - 동기 및 비동기 이벤트
  - EventEmitter vs 콜백
  - 콜백과 EventEmitter의 결합



## 도입

- 우리는 동기식 프로그래밍에서 특정 문제를 해결하기 위해 정의한 일련의 연속적 연산 단계들로 코드를 생각하는 것에 익숙합니다.
  - 모든 작업은 블로킹입니다.
  - 즉, 현재 작업이 완료됐을 때 다음 작업을 실행할 수 있습니다.
  - 이 방법을 사용하면 코드를 쉽게 이해하고 디버깅할 수 있습니다.
- 반면에, 비동기식 프로그래밍에서는 파일 읽기 또는 네트워크 요청 수행과 같은 일부 작업을 백그라운드 작업으로 실행할 수 있습니다.
  - 비동기 작업이 호출되면 이전 작업이 아직 완료되지 않은 경우에도 다음 작업이 즉시 실행됩니다.
  - 이 상황에서는 비동기 작업이 끝났을 때 이를 통지받아 해당 작업의 결과를 사용하여 다음의 작업을 이어나가야 합니다.
- Node.js에서 비동기 작업의 완료를 통지받는 가장 기본적인 메커니즘은 **콜백**입니다.
  - 콜백은 비동기 작업의 결과를 가지고 런타임에 의해 호출되는 함수일 뿐입니다.



- 콜백은 다른 모든 비동기 메커니즘을 기초로 하는 것들의 가장 기본적인 구성요소입니다.
  - 실제로 콜백 없이는 프라미스가 존재할 수 없으며 따라서 async/await 또한 존재할 수 없습니다.
  - 또한 스트림이나 이벤트 또한 불가능합니다.
  - 이것이 콜백이 어떻게 작동하는지 알아야 하는 이유입니다.



## 3-1 콜백 패턴

- 콜백은 이전 장에서 소개한 리액터 패턴의 **핸들러**를 구현한 것입니다.
  - Node.js에 독특한 프로그래밍 스타일을 제공하는 상징 중 하나입니다.
- 콜백은 작업의 결과를 전달하기 위해서 호출되는 함수이며, 우리가 비동기 작업을 처리할 때 필요한 것입니다.
  - 비동기 세계에서 콜백은 동기적으로 사용되는 `return` 명령의 사용을 대신합니다.
- JS는 콜백에 이상적인 언어입니다.
  - 그 이유는 함수가 일급 클래스 객체(first class object)이면서 변수에 할당하거나 인자로 전달되거나 다른 함수 호출에서 반환되거나 자료구조에 저장될 수 있기 때문입니다.
- 콜백을 구현하는 또 다른 이상적인 구조는 **클로저(closure)**입니다.
  - 클로저를 사용하면 실제로 생성된 함수의 환경을 참조할 수 있습니다.
  - 콜백이 언제 어디서 호출되는지에 관계없이 비동기 작업이 요청된 컨텍스트를 항상 유지할 수 있기 때문입니다.

- 이번 섹션에서는 `return` 명령을 대신하여 콜백으로 이루어진 프로그래밍 스타일들을 분석해 볼 것입니다.



### 3-1-1 연속 전달 방식

- JS에서 **콜백**은 다른 함수에 인자로 전달되는 함수이며, 작업이 완료되면 작업 결과를 가지고 호출됩니다.
- 함수형 프로그래밍에서 이런 식으로 결과를 전달하는 방식을 **연속 전달 방식(CPS: continuation-passing style)**이라고 합니다.
  - 이는 일반적인 개념이며, 항상 비동기 작업과 관련있는 것은 아닙니다.
  - 사실, 단순히 결과를 호출자에게 직접 반환하는 대신 결과를 다른 함수(콜백)로 전달하는 것을 말합니다.



#### 동기식 연속 전달 방식

- 개념을 명확히 하기 위해 간단한 동기 함수를 살펴봅시다.

```js
function add(a, b) {
	return a + b;
}
```

- 결과는 `return` 문을 통해 호출자에게 전달됩니다.
- 이것을 **직접 스타일(direct style)**이라고 하며, 동기식 프로그래밍에서 일반적으로 결과를 반환하는 방식을 보여줍니다.



- 앞의 함수와 동일한 처리를 CPS로 바꾼 코드는 다음과 같습니다.

```js
function addCps(a, b, callback) {
	callback(a + b);
}
```



- `addCps()` 함수는 동기 CPS 함수로 콜백 또한 작업이 완료되었을 때 작업을 완료합니다.
- 다음의 코드는 이를 증명합니다.

```js
console.log('before');
addCps(1, 2, result => console.log(`Result: ${result}`));
console.log('after')
```



- `addCps()`가 동기적이기 때문에 앞선 코드는 다음과 같이 순서대로 출력됩니다.

```
before
Result: 3
after
```





#### 비동기 연속 전달 방식

- `addCps()` 함수가 비동기인 경우를 생각해봅시다.

```js
function addAsync(a, b, callback) {
  setTimeout(() => callback(a + b), 100);
}
```

- 앞의 코드에서 `setTimeout()`을 사용하여 콜백의 비동기 호출을 가정해보았습니다.
- `setTimeout()`은 이벤트 큐에 주어진 밀리초 후에 실행되는 작업을 추가합니다.
- 이는 명백한 비동기 작업입니다.



- 이제 `addAsync()`를 사용하여 작업의 순서가 어떻게 변경되는지 살펴보겠습니다.

```js
console.log('before')
addAsync(1, 2, result => console.log(`Result: ${result}`))
console.log('after')
```



- 앞의 코드는 다음과 같은 결과를 출력합니다.

```
before
after
Result: 3
```



- `setTimeout()`은 비동기 작업을 실행시키기 때문에 콜백의 실행이 끝날 때까지 기다리지 않는 대신, 즉시 반환되어 `additionAsync()`로 제어를 돌려주어 제어가 호출자에게 반환됩니다.
- Node.js의 이 속성은 매우 중요한데, 그 이유는 비동기 요청이 전달된 후 즉시 제어를 이벤트 루프에 돌려주고 큐(대기열)에 있는 새로운 이벤트가 처리될 수 있도록 하기 때문입니다.



**[그림 3.1 비동기 함수 호출의 제어 흐름]**

![image-20220102133213090](/Users/mojo/Library/Application Support/typora-user-images/image-20220102133213090.png)



- 비동기 작업이 완료되면 실행은 비동기 함수에 제공된 콜백에서부터 실행이 재개됩니다.
- 실행은 이벤트 루프에서 시작되기 때문에 새로운 스택을 갖습니다.
  - 이 부분이 JS가 정말 유용한 지점입니다.
  - 클로저 덕분에 콜백이 다른 시점과 다른 위치에서 호출되더라도 비동기 함수의 호출자 컨텍스트를 유지하기 때문입니다.



- 정리해보면, 동기 함수는 조작을 완료할 때까지 블로킹합니다.
  - 비동기 함수는 제어를 즉시 반환하고 결과는 이벤트 루프의 다음 사이클에서 핸들러(이 경우에는 콜백)로 전달됩니다.



#### 비 연속 전달(Non-CPS) 콜백

- 콜백 인자가 있는 경우들에는 함수가 비동기식이거나 연속 전달 스타일(CPS)을 사용한다고 생각하게 할 수 있는 상황들이 있습니다.
  - 그러나 항상 그런 것은 아닙니다.
- 예를 들어 Array 객체의 `map()` 함수를 살펴보겠습니다.

```js
const result = [1, 5, 7].map(element => element - 1);
console.log(result) // [0, 4, 6]
```



- 콜백은 배열 내의 요소를 반복하는데 사용될 뿐 연산 결과를 전달하지 않습니다.
  - 실제로 여기서 결과는 직접적인 방식으로 동기적으로 반환됩니다.
- 연속 전달 방식과 비 연속 전달 방식 사이에는 문법적 차이가 없습니다.
- 따라서 콜백의 목적은 API 문서에 분명하게 명시됩니다.



### 3-1-2 동기? 비동기?

- 우리는 함수가 동기식인지 또는 비동기식인지의 특성에 따라 실행 순서가 어떻게 완전히 변화하는지 살펴보았습니다.
  - 이것은 정확성과 효율성 모든 면에서 전체 애플리케이션의 흐름에 많은 영향을 미칩니다.
- 이제 두 가지 패러다임과 위험에 대한 분석을 해보겠습니다.
- 일반적으로 반드시 피해야 할 것은 API의 이러한 특성과 관련하여 발견하기 어렵고 재현 불가능한 문제를 일으키는 모순과 혼돈을 만드는 것입니다.
- 분석을 진행하기 위해, 일관성없는 비동기 함수의 경우를 예를 들어 설명하겠습니다.



#### 예측할 수 없는 함수

- 가장 위험한 상황 중 하나는 특정 조건에서 동기적으로 동작하고 다른 조건에서는 비동기적으로 동작하는 API를 갖는 것입니다.
- 다음 코드를 예로 들어보겠습니다.

```js
import { readFile } from 'fs'

const cache = new Map()

function inconsistentRead(filename, cb) {
  if(cache.has(filename)) {
    // 동기적으로 호출됨
   cb(cache.get(filename))
  } else {
    // 비동기 함수
    readFile(filename, 'utf8', (err, data) => {
      cache.set(filename, data)
      cb(data)
    })
  }
}
```

- 앞의 함수는 cache 변수를 사용하여 서로 다른 파일을 읽어 작업의 결과를 저장합니다.
- 이것은 예제일 뿐이며, 오류 관리도 없고 캐싱 로직("11장. 고급 레시피"에서 비동기 캐싱을 어떻게 적절히 다루는지 배울 것입니다) 자체가 꼭 이렇게 되어야 하는 것이 아님을 명심하세요.
- 위의 함수는 파일이 처음 읽혀지고 캐싱될 때까지는 비동기적으로 동작하지만 캐시에 이미 있는 파일에 대한 모든 후속 요청에 대해서는 동기적으로 동작하여 즉각적으로 콜백을 호출하므로 매우 위험합니다.



#### Zalgo를 풀어놓다

- 이제 앞서 정의한 것과 같이 예측할 수 없는 함수를 사용하면 애플리케이션이 쉽게 손상될 수 있다는 것을 살펴보겠습니다.

```js
function createFileReader(filename) {
  const listeners = []
  inconsistentRead(filename, value => {
    listeners.forEach(listener => listener(value))
  })
  
  return {
    onDataReady: listener => listeners.push(listener)
  }
}
```

- 앞의 함수가 호출됐을 때, 통지(notifier) 역할을 하는 새로운 객체를 생성하며 파일 읽기 작업에 대해 다수의 리스너를 설정할 수 있게 해줍니다.
- 읽기 작업이 완료되고 데이터를 사용할 수 있게 될 때 모든 리스너가 한번에 호출됩니다.
- 위의 함수는 앞서 만든 `inconsistentRead()` 함수를 사용하여 이 기능을 구현합니다.
- 이제 `createFileReader()` 함수를 사용해보겠습니다.

```js
const reader1 = createFileReader('data.txt')
reader1.onDataReady(data => {
  console.log(`First call data: ${data}`)
  
  // 얼마 후 같은 파일을 다시 읽으려고 시도합니다.
 	const reader2 = createFileReader('data.txt')
  reader2.onDataReady(data => {
    console.log(`Second call data: ${data}`)
  })
})
```

- 이 코드는 다음과 같은 결과를 출력합니다.

```
First call data: some data
```

- 출력에서 볼 수 있듯 두 번째 콜백이 호출되지 않습니다. 왜 그런지 보겠습니다.
  - `reader1`이 생성되는 동안 `inconsistentRead()` 함수는 사용 가능한 캐시된 결과가 없으므로 비동기적으로 동작합니다.
    - 따라서 우리는 리스너를 등록하는데 충분한 시간을 가질 수 있습니다. 
    - 읽기 작업이 완료된 후, 나중에 이벤트 루프의 다른 사이클에서 리스너가 호출되기 때문입니다.
  - 그런 다음, `reader2`는 요청된 파일에 대한 캐시가 이미 존재하는 이벤트 루프의 사이클에서 생성됩니다.
    - 이 경우는 `inconsistentRead()`에 대한 내부 호출은 동기 방식이 됩니다.
    - 따라서 콜백은 즉시 호출됩니다.
    - 즉, `reader2`의 모든 리스너들이 동기적으로 호출됩니다.
    - 하지만 우리는 리스너를 `reader2`의 생성 후에 등록하기 때문에 이들이 호출되는 일은 결코 발생하지 않는 것입니다.

- `inconsistentRead()` 함수의 콜백 동작은 실제로 호출 빈도, 인자로 전달되는 파일명 및 파일을 읽어들이는 데 걸리는 시간과 같은 여러 요인에 의해 달리자므로 실제로 예측할 수 없습니다.



- 우리가 방금 본 버그는 실제 애플리케이션에서 식별하고 재현하는 것이 매우 어려울 수 있습니다.
  - 동시에 여러 요청이 존재할 수 있는 웹 서버에서 유사한 기능을 사용한다고 가정해봅시다.
  - 명백한 이유도 없이 어떠한 오류도, 로그도 없이 처리되지 않는 요청이 발생한다고 상상해보세요. 이것은 확실히 위험한 결함입니다.



- Zalgo란 세계의 광기, 죽음, 파괴를 일으키리라 믿어지는 불길한 존재에 대한 인터넷 상의 전설입니다.



#### 동기 API의 사용

- Zalgo의 사례에서 알 수 있는 교훈은 API의 동기 또는 비동기 특성을 명확하게 정의하는 것이 필수적이라는 것입니다.
- `inconsistentRead()` 함수를 적절하게 수정할 수 있는 방법 중 한가지는 완전히 동기화시키는 것입니다.
  - 이것은 Node.js가 대부분의 기본 I/O 작업에 대한 동기식 직접 스타일 API 셋을 제공하기 때문에 가능합니다.
- 예를 들어, 비동기 형식 대신 `fs.readFileSync()` 함수를 사용할 수 있는데, 코드는 다음과 같습니다.

```js
import { readFileSync } from 'fs'

const cache = new Map()

function consistentReadSync(filename) {
  if(cache.has(filename)) {
    return cache.get(filename)
  } else {
    const data = readFileSync(filename, 'utf8')
    cache.set(filename, data)
    return data
  }
}
```

- 전체 기능이 직접 스타일로 변환되었음을 알 수 있습니다. 함수가 동기식이면 함수가 CPS를 가질 이유가 없습니다.
- 실제로 직접 스타일을 사용하여 동기식 API를 구현하는 것이 항상 최선의 방법이라고 말할 수 있습니다.
- 이는 애플리케이션을 둘러싼 환경의 혼란을 제거하고 성능 측면에서 보다 효율적일 것입니다.



> **패턴**
>
> 순수한 동기식 함수에 대해서는 직접 스타일을 사용하세요.



- CPS에서 직접 스타일로 혹은 비동기에서 동기로 또는 그 반대로 API를 변경하면 API를 사용하는 모든 코드의 스타일을 변경해야 할 수도 있습니다.
- 일례로, 우리의 경우 `createFileReader()` API의 인터페이스를 완전히 변경하고 항상 동기적으로 동작하도록 수정해야 합니다.



- 또한 비동기 API 대신 동기 API를 사용하면 몇 가지 주의해야 할 사항이 있습니다.
  - 특정 기능에 대한 동기식 API를 항상 사용할 수 있는 것은 아닙니다.
  - 동기 API는 이벤트 루프를 블록하고 동시 요청을 보류합니다. JS 동시성 모델을 깨뜨려서 전체 애플리케이션 속도를 떨어뜨립니다.



- `consistentReadSync()` 함수에서 동기식 I/O API는 하나의 파일당 한번의 호출이 일어나고 이후의 호출에는 캐시에 저장된 값을 사용하기 때문에, 이벤트 루프를 블로킹하는 위험은 부분적으로 완화됩니다.
- 제한된 수의 정적 파일로 작업을 할 경우에는 `consistentReadSync()`를 사용하는 것이 이벤트 루프에 큰 영향을 미치지 않습니다.
- 하지만 한 번이라도 큰 파일을 읽는 경우라면 얘기가 완전히 달라집니다.



- Node.js에서 동기 I/O를 사용하는 것은 많은 경우에 권장되지 않습니다.
- 그러나 어떤 경우에는 그것이 가장 쉽고 효율적인 해결책이 되기도 합니다.
  - 예를 들어, 애플리케이션이 부팅되는 동안 동기적 차단 API를 사용하여 환경 파일들을 로드하는 것이 최선입니다.



> **패턴**
>
> 애플리케이션이 비동기적 동시성 작업을 처리하는데 영향을 주지 않는 경우에만 블로킹 API를 사용하세요.





#### 지연 실행(deferred execution)으로 비동시성을 보장

- `inconsistentRead()` 함수를 수정하는 또 다른 방법은 완전한 비동기로 만드는 것입니다.
  - 여기서의 트릭은 동기 콜백 호출이 동일한 이벤트 루프 사이클에서 즉시 실행되는 대신 "가까운 미래"에 실행되도록 예약하는 것입니다.
  - Node.js에서는 `process.nextTick()`을 사용하여 이 작업을 수행할 수 있습니다.
- `process.nextTick()`은 현재 진행 중인 작업의 완료 시점 뒤로 함수의 실행을 지연시킵니다.
  - 그 기능은 매우 간단합니다.
  - 콜백을 인수로 취하여 대기 중인 I/O 이벤트 대기열의 앞으로 밀어 넣고 즉시 반환합니다.
  - 그렇게 되면 현재 진행중인 작업이 제어를 이벤트 루프로 넘기는 즉시 콜백이 실행됩니다.
- 이 기술을 적용하여 `inconsistentRead()` 함수를 다음과 같이 수정합니다.

```js
import { readFile } from 'fs'

const cache = new Map()

function consistentReadAsync(filename, callback) {
  if(cache.has(filename)) {
    // 지연된 콜백 호출
    process.nextTick(() => callback(cache.get(filename)))
  } else {
    // 비동기 함수
    readFile(filename, 'utf8', (err, data) => {
      cache.set(filename, data)
      callback(data)
    })
  }
}
```

- 이제 함수는 어떤 상황에서도 콜백을 비동기적으로 호출할 수 있게 되었습니다.
- `inconsistentRead()` 함수 대신에 이것을 사용하고 Zalgo가 없다는 것을 확실하게 해두는 게 좋습니다.



>  **패턴**
>
> `process.nextTick()`을 사용하여 실행을 연기함으로써 콜백의 비동기적 호출을 보장할 수 있습니다.



- 코드의 실행을 지연시키는 또 다른 API는 `setImmediate()`입니다. `process.nextTick()`과 목적은 유사하지만 그 의미는 크게 다릅니다. 
  - `process.nextTick()`으로 지연된 콜백은 **마이크로태스크**라 불리며, 그것들은 현재의 작업이 완료된 후에 바로 실행되며 다른 I/O 이벤트가 발생하기 전에 실행됩니다.
  - 반면에 `setImmediate()`는 이미 큐에 있는 I/O 이벤트들의 뒤에 대기하게 됩니다.
  - `process.nextTick()`은 이미 예정된 I/O보다 먼저 실행되기 때문에 재귀 호출과 같은 특정 상황에서 **I/O 기아(starvation)**를 발생시킬 수 있습니다.
  - `setImmediate()`에서는 이런 일이 절대 일어나지 않습니다.



- `setTimeout(callback, 0)`은 `setImmediate()`와 비슷한 동작을 가집니다.
- 하지만 특정 상황에서는 `setImmediate()`로 예약된 콜백이 `setTimeout(callback, 0)`으로 예약된 것보다 빨리 실행됩니다.
- 그 이유를 보자면 이벤트 루프가 모든 콜백을 각기 다른 단계에서 실행시킨다는 것을 고려해야 합니다.
- 우리는 I/O 콜백 전에 실행되는 타이머(`setTimeout()`)를 가지고 있습니다.
- 그것은 `setImmediate()` 콜백 이전에 실행됩니다.
- 즉, `setTimeout()` 콜백 안에서, I/O 콜백 안에서 또는 이 두 단계 이후에 큐에 들어가는 마이크로태스크 안에서 `setImmediate()`로서 큐에 작업을 넣게 되면 현재 우리가 있는 단계 바로 이후에 오는 단계에서 콜백이 실행됩니다.
- `setTimeout()` 콜백은 이벤트 루프의 다음 사이클을 기다려야 합니다.



- 이후에 이 책에서 동기적 CPU 바운딩 작업 실행을 위한 지연 호출의 사용을 파악해 나갈 때, 이 API 사이의 차이점에 대해서 더 나은 이해를 하게 될 것입니다.



### 3-1-3 Node.js 콜백 규칙

- Node.js에서 CPS API 및 콜백은 일련의 특정한 규칙을 따릅니다.
- 이 규칙은 Node.js 코어 API에 적용되지만 대다수의 사용자 영역 모듈과 애플리케이션에도 적용됩니다.
- 따라서 콜백이 사용되는 비동기 API를 설계할 때마다 이를 이해하고 반드시 준수해야 합니다.



#### 콜백은 맨 마지막에

- 모든 Node.js 코어 함수에서 표준 규칙은 함수가 입력으로서 콜백을 허용한다면 콜백이 맨 마지막 인자로 전달되어야 한다는 것입니다.
- 다음 Node.js 코어 API를 예로 들어 보겠습니다.

```js
readFile(filename, [options], callback)
```

- 이 함수의 특성에서 볼 수 있듯이 여러 인자가 있는 경우에도 콜백은 항상 마지막 위치에 놓입니다.
- 이 규칙이 존재하는 이유는 콜백이 적절한 위치에 정의되어 있는 경우, 함수 호출의 가독성이 더 좋기 때문입니다.



#### 오류는 맨 처음에

- CPS에서 오류가 다른 유형의 결과처럼 전달되므로 콜백의 사용이 필요합니다.
- Node.js에서 CPS 함수에 의해 생성된 **오류**는 항상 콜백의 <u>첫 번째 인자</u>로 전달되며, 실제 **결과**는 <u>두 번째 인자</u>에서부터 전달됩니다.
  - 동작이 에러 없이 성공하였을 때, 첫 번째 인자는 `null` 또는 `undefined`가 됩니다.
- 다음의 코드는 이 규칙을 준수하는 콜백을 정의하는 방법을 보여주고 있습니다.

```js
readFile('foo.txt', 'utf8', (err, data) => {
  if(err) {
    handleError(err)
  } else {
    processData(data)
  }
})
```

- 에러가 있는지를 항상 체크하는 것이 좋습니다.
- 그렇지 않으면 코드를 디버깅하고 에러 지점을 찾는 것이 어려울 수 있습니다.
- 고려해야 할 또 다른 중요한 규칙은 오류는 항상 `Error` 타입이어야 한다는 것입니다.
  - 즉, 간단한 문자열이나 숫자를 오류 객체로 전달해서는 안됩니다.



#### 오류 전파

- 동기식 직접 스타일 함수의 오류 전파는 잘 알려진 `throw`문을 사용해 수행되므로 오류가 `catch` 될 때까지 호출 스택에서 실행됩니다.
- 그러나 비동기식 CPS에서 적절한 에러 전파는 오류를 호출 체인의 다음에서 콜백으로 전달하여 수행됩니다.
- 일반적인 패턴은 다음과 같습니다.

```js
import { readFile } from 'fs'

function readJSON(filename, callback) {
  readFile(filename, 'utf8', (err, data) => {
    let parsed
    
    if(err) {
      // 에러를 전파하고 현재의 함수에서 빠져 나옴
      return callback(err)
    }
    
    try {
      // 파일 내용 파싱
      parsed = JSON.parse(data)
    } catch (err) {
      // 파싱 에러 캐치
      return callback(err)
    }
    
    // 에러 없음, 데이터 전파
    callback(null, parsed)
  })
}
```

- 주목해야 할 점은 `readFile()`을 수행하였을 때의 에러결과를 어떻게 전파하는지입니다.
  - 에러를 다시 밖으로 발생시키거나 리턴하지 않습니다.
  - 대신에 다른 결과처럼 단지 콜백을 사용합니다.
- 또한 우리는 `JSON.parse()`가 발생시키는 에러를 포착하기 위해서 `try...catch` 구문을 사용하였습니다.
  - 해당 함수는 동기식 함수이므로 호출자에게 에러를 전달하기 위해서 전통적인 `throw`를 사용합니다.
  - 마지막으로 모든 것이 잘 작동하였을 경우 콜백은 에러가 없다는 것을 나타내기 위해서 첫 번째 인자로 `null`과 함께 호출됩니다.



- 우리가 `try` 블럭 내에서 콜백 호출을 피하고 있는 방식 또한 흥미로운 부분입니다.
  - 콜백이 자체적으로 발생시키는 에러를 포착하는 것은 우리가 원하는 것이 아니기 때문입니다.



#### 캐치되지 않는 예외

- 때때로 비동기 함수의 콜백 내에서 에러는 밖으로 전달되거나 포착되지 않는 상황이 발생합니다.
  - 우리가 앞서 정의한 `readJSON()` 함수 내에서 `try ... catch` 구문으로 `JSON.parse()`를 둘러싸지 않는다면 그런 상황이 발생할 수 있습니다.
- 비동기식 콜백 내부에서 예외를 발생시키는 것은 예외가 이벤트 루프로 이동하게 만들며, 이것은 절대 다음 콜백으로 전파되지 않게 됩니다.
  - Node.js에서는 이것이 회복 불능 상태이며 애플리케이션은 0이 아닌 종료 코드와 함께 그냥 종료되고 stderr 인터페이스를 통해 오류를 출력합니다.
- 이를 재현해 보기 위해 앞서 `readJSON()` 함수에 정의했던 `JSON.parse()`를 둘러싼 `try ... catch` 블럭을 제거해보겠습니다.

```js
function readJSONThrows(filename, callback) {
  readFile(filename, 'utf8', (err, data) => {
    if (err) {
      return callback(err)
    }
    
    callback(null, JSON.parse(data))
  })
}
```

- 이제 방금 정의한 함수에서는 `JSON.parse()`에서 발생하는 예외를 잡을 방법이 없습니다.
- 다음과 같은 코드로 부적합한 JSON 파일을 파싱한다면,

```js
readJSONThrows('invalid_json.json', (err) => console.error(err))
```

- 애플리케이션이 갑자기 종료되고 콘솔에 다음과 같은 예외 메세지가 출력됩니다.

```shell
SyntaxError: Unexpected token h in JSON at position 1
 at JSON.parse (<anonymous>)
 at file:///.../03-callbacks-and-events/08-uncaught-errors/index.js:8:25
 at FSReqCallback.readFileAfterClose [as oncomplete] (internal/fs/read_file_context.js:61:3)
```

- 앞의 스택 트레이스를 살펴보면 내장 `fs` 모듈에서 시작하여, 정확히 네이티브 API가 읽기를 완료한 후 이벤트 루프를 통해 `fs.readFile()` 함수로 그 결과를 반환한 지점으로부터 시작됩니다.
- 이것은 명확히 예외가 콜백에서 스택으로 이동한 다음, 즉시 이벤트 루프로 이동하여 마지막으로 콘솔에서 포착되어 `throw` 된다는 것을 보여줍니다.



- 이것은 `readJSONThrows()`를 `try ... catch` 블럭으로 둘러싸서 호출한다고 하더라도 블럭이 동작하는 스택과 콜백이 호출된 스택이 다르기 때문에 동작하지 않는다는 것을 의미합니다.
- 다음 코드는 방금 설명된 것에 대한 안티 패턴을 보여줍니다.

```js
try {
	readJSONThrows('invalid_json.json', (err) => console.log(err))
} catch (err) {
  console.log('This will NOT catch the JSON parsing exception')
}
```

- 위의 `catch` 구문은 JSON 파싱 에러를 절대로 받을 수 없게 됩니다.
  - 에러는 비동기식 실행을 발생시키는 함수 안에서가 아니고 이벤트 루프에 예외가 발생한 별도의 콜 스택을 타고 올라갑니다.
- 앞서 언급한 것처럼, 예외가 이벤트 루프에 도달하는 순간 애플리케이션은 중단됩니다.
- 그러나 우리는 애플리케이션이 중단되기 이전에 자원을 정리하거나 로그를 남길 수 있습니다.
- 실제로 이러한 경우에, Node.js는 프로세스를 종료하기 직전에 `uncaughtException`이라는 특수 이벤트를 내보냅니다.
- 다음 코드는 이러한 경우에 사용하는 코드의 예시입니다.

```js
process.on('uncaughtException', (err) => {
  console.error(`This will catch at last the JSON parsing exception: ${err.message}`)
  // 종료 코드 1(에러)과 함께 애플리케이션 종료
  // 아래의 코드가 없으면 애플리케이션은 계속됨
  process.exit(1)
})
```

- 캐치되지 않는 예외가 애플리케이션의 일관성을 보장할 수 없는 상태로 만듭니다.
  - 이로 인해 예기치 않는 문제가 발생할 수 있음을 이해하는 것이 중요합니다.
- 예를 들어, 여전히 불완전한 I/O 요청이 실행 중이거나 클로저가 일치하지 않을 수 있습니다.
- 캐치되지 않은 예외가 발생한 경우, 특히 OS에서는 애플리케이션을 실행상태에 두지 않는 것이 권장됩니다.
- 선택적으로 필요한 작업의 정리 후에 프로세스는 즉시 종료되어야 하며, 프로세스 관리자가 애플리케이션을 재시작해야 합니다.
- 이것은 **fail-fast** 접근법으로 알려져 있고 Node.js에서 권장되는 사항입니다.



- 이것으로 콜백 패턴의 소개를 마무리 짓겠습니다.
- 이제 Node.js와 같은 이벤트 기반 플랫폼에서의 또 다른 주요 구성요소인 관찰자 패턴을 살펴보도록 하겠습니다.



## 3-2 관찰자 패턴 (The observer pattern)

- Node.js에서 기본적으로 사용되고 중요한 또 다른 패턴은 **관찰자 패턴**입니다.
  - 리액터(Reactor) 그리고 콜백(Callback)과 함께 **관찰자 패턴**은 비동기적인 Node.js 세계를 숙달하는 데 필수적인 조건입니다.
- 관찰자 패턴은 Node.js의 반응적(reactive) 특성을 모델링하고 콜백을 완벽하게 보완하는 이상적인 해결책입니다.
- 다음과 같이 공식적인 정의를 내릴 수 있습니다.
  - **관찰자 패턴**은 상태 변화가 일어날 때 관찰자(또는 listener)에게 통지할 수 있는 객체를 정의하는 것입니다.
- 콜백 패턴과 가장 큰 차이점은 전통적인 CPS 콜백이 일반적으로 오직 <u>하나</u>의 리스너에게 결과를 전달하는 반면, 관찰자 패턴은 주체가 실질적으로 <u>여러</u> 관찰자에게 통지를 할 수 있다는 점입니다.



### 3-2-1 EventEmitter 클래스

- 전통적인 객체지향 프로그래밍에서는 관찰자 패턴에 인터페이스, 구체적인 클래스 그리고 계층구조를 요구하지만 Node.js에서는 훨씬 더 간단합니다.
- 관찰자 패턴은 이미 코어에 내장되어 있으며, EventEmitter 클래스를 통해 사용할 수 있습니다.
- EventEmitter 클래스를 사용하여 특정 유형의 이벤트가 발생하면 호출되는 하나 이상의 함수를 리스너로 등록할 수 있습니다.



**[그림 3.2 EventEmitter로부터 이벤트를 받는 리스너]**

![image-20220102192153562](/Users/mojo/Library/Application Support/typora-user-images/image-20220102192153562.png)



- EventEmitter는 `events` 코어 모듈로부터 export됩니다. 다음 코드는 EventEmitter에 대한 참조를 얻을 수 있는 방법을 보여줍니다.

```js
import { EventEmitter } from 'events'
const emitter = new EventEmitter()
```



- EventEmitter의 필수 메서드는 다음과 같습니다.
  - `on(event, listener)`: 이 메서드를 사용하면 주어진 이벤트 유형(문자열)에 대해 새로운 리스너(함수)를 등록할 수 있습니다.
  - `once(event, listener)`: 이 메서드는 첫 이벤트가 전달된 후 제거되는 새로운 리스너를 등록합니다.
  - `emit(event, [arg1], [...])`: 이 메서드는 새 이벤트를 생성하고 리스너에게 전달할 추가적인 인자들을 제공합니다.
  - `removeListener(event, listener)`: 이 메서드는 지정된 이벤트 유형에 대한 리스너를 제거합니다.

- 앞의 모든 메서드들은 연결(chaining)이 가능하게 하기 위해 EventEmitter 인스턴스를 반환합니다.
- `listener` 함수는 시그니처 함수([arg1], [...])를 가지고 있기 때문에 이벤트가 발생된 순간에 전달되는 인수들을 쉽게 받아들일 수 있습니다.



- 여러분은 이미 리스너와 전통적인 Node.js 콜백 간의 큰 차이점이 있다는 것을 보았습니다.
- 첫 번째 인자가 꼭 에러일 필요는 없으며, `emit()`이 호출될 때 어떤 값이든 전달 가능합니다.



### 3-2-2 EventEmitter 생성 및 사용

- 실제로 EventEmitter를 어떻게 사용할 수 있는지 살펴보도록 하겠습니다.
  - 가장 간단한 방법은 새로운 인스턴스를 만들어 바로 사용하는 것입니다.
- 다음 코드는 EventEmitter를 사용하여 파일 목록에서 특정 패턴이 발견되면 실시간으로 구독자들에게 통지를 하는 함수를 보여줍니다.

```js
import { EventEmitter } from 'events'
import { readFile } from 'fs'

function findRegex(files, regex) {
  const emitter = new EventEmitter()
  for (const file of files) {
    readFile(file, 'utf8', (err, content) => {
      if (err) {
        return emitter.emit('error', err)
      }
      
      emitter.emit('fileread', file)
      const match = content.match(regex)
      if (match) {
        match.forEach(elem => emitter.emit('found', file, elem))
      }
    })
  }
  
  return emitter
}
```

- 우리가 방금 정의한 함수는 3가지의 이벤트를 발생시키는 EventEmitter 인스턴스를 반환합니다.
  - 'fileread', 파일을 읽을 때
  - 'found', 일치하는 항목이 발견되었을 때
  - 'error', 파일을 읽는 동안 에러가 발생하였을 때



- `findRegex()` 함수가 어떻게 사용되는지 보겠습니다.

```js
findRegex(
	['fileA.txt', 'fileB.json'],
  /hello \w+\g
)
	.on('fileread', file => console.log(`${file} was read`))
	.on('found', (file, match) => console.log(`Matched "${match}" in ${file}`))
	.on('error', err => console.error(`Error emitted ${err.message}`))
```

- `findRegex()` 함수에서 생성한 EventEmitter에 의해 발생되는 3가지 이벤트 타입에 각각 리스너를 등록하였습니다.



### 3-2-3 오류 전파

- EventEmitter는 콜백에서처럼 에러가 발생하였을 때 예외를 단지 `throw`할 수 없습니다.
- 대신 `error`라는 특수한 이벤트를 발생시키고, `Error` 객체를 인자로 전달하는 규약을 따릅니다.
- 이것이 정확히 우리가 앞에서 정의한 `findRegex()` 함수가 하고 있는 것입니다.



> EventEmitter는 error 이벤트를 특별한 방법으로 다룹니다. 
>
> 만약 해당 이벤트가 발생하고 관련된 리스너가 없을 경우에 자동으로 예외를 throw하고 애플리케이션을 빠져 나옵니다.
>
>  이러한 이유로 error 이벤트에 대한 리스너를 항상 등록해주는 것이 권장됩니다.



### 3-2-4 관찰 가능한 객체 만들기

- 앞선 예제에서 보았듯이 Node.js 세계에서는 EventEmitter 자체로 사용되는 경우는 매우 드뭅니다.
  - 대신, 다른 클래스의 확장이 일반적입니다.
- 실제로 어떤 클래스라도 EventEmitter의 기능을 상속받아 관찰 가능한 객체가 되는 것이 가능합니다.
- 이 패턴을 설명하기 위해서 다음과 같이 `findRegex()` 함수의 기능을 구현해 보겠습니다.

```js
import { EventEmitter } from 'events'
import { readFile } from 'fs'

class FindRegex extends EventEmitter {
  constructor(regex) {
    super()
    this.regex = regex
    this.files = []
  }
  
  addFile(file) {
    this.files.push(file)
    return this
  }
  
  find() {
    for (const file of this.files) {
      readFile(file, 'utf8', (err, content) => {
        if (err) {
          return this.emit('error', err)
        }
        
        this.emit('fileread', file)
        
        const match = content.match(this.regex)
        if (match) {
          match.forEach(elem => this.emit('found', file, elem))
        }
      })
    }
  }
}
```

- 우리가 정의한 findRegex 클래스는 EventEmitter를 extends 하여 완전한 관찰가능 객체가 되었습니다.
- EventEmitter 내부 구성을 초기화하기 위해서 constructor 내부에서 항상 super()를 사용하는 것을 기억하세요.



- 다음은 우리가 방금 정의한 `FindRegex()` 클래스를 어떻게 사용하는지에 대한 예제입니다.

```js
const findRegexInstance = new FindRegex(/hello \w+/)
findRegexInstance
	.addFile('fileA.txt')
	.addFile('fileB.json')
	.find()
	.on('found', (file, match) => console.log(`Matched "${match}" in file ${file}`))
	.on('error', err => console.error(`Error emitted ${err.message}`))
```

- FindRegex 객체가 EventEmitter로부터 상속받은 `on()` 메서드를 어떻게 제공하는지 확인할 수 있습니다.
  - 이것은 Node.js 생태계에서 꽤나 일반적인 패턴입니다.
- 예를 들어, 핵심 HTTP 모듈의 Server 객체의 EventEmitter 함수 상속은 request(새로운 요청을 받았을 때), connection(새로운 연결이 성립되었을 때), closed(서버 소켓이 닫혔을 때)와 같은 이벤트를 생성하게끔 합니다.
- EventEmitter를 확장하는 주목할 만한 또 다른 객체의 예는 Node.js 스트림입니다. 6장에서 상세하게 다룹니다.



### 3-2-5 EventEmitter와 메모리 누수

- 관찰 가능 주체들에 대해서 오랜 시간동안 구독을 하고 있을 때 더 이상 그것들이 필요하지 않게 되면 **구독 해지**하는 것이 매우 중요합니다.
- 구독 해지는 **메모리 누수**를 예방하고 리스너의 스코프에 있는 객체에 의해 더 이상 사용되지 않는 메모리 점유를 풀게 해줍니다.
- Node.js에서는 EventEmitter 리스너의 등록을 해지하지 않는 것이 메모리 누수의 주요 원인이 됩니다.



- 메모리 누수는 메모리가 더 이상 필요하지 않지만 해제되지 않아 애플리케이션의 메모리 사용을 무기한으로 증가시키는 원인을 제공하는 소프트웨어 결함입니다.
- 예를 들어 다음과 같은 코드를 생각해 볼 수 있습니다.

```js
const thisTakesMemory = 'A big string....'
const listener = () => {
  console.log(thisTakesMemory)
}
emitter.on('an_event', listener)
```

- 변수 `thisTakesMemory`는 리스너에서 참조되며 리스너가 해제되기 전까지 또는 emitter 자체에 대한 참조가 더 이상 활성화되어 있지 않아 도달할 수 없게 되어 가비지 컬렉션에 잡히기 전까지는 메모리에서 유지됩니다.



- 즉, 애플리케이션의 전체 주기 동안 EventEmitter는 그것의 모든 리스너들이 참조하는 메모리와 함께 도달 가능한 상태를 유지한다는 것을 의미합니다.
  - 예를 들어, 우리가 HTTP의 요청이 올 때마다 해제되지 않는 "영구적인" EventEmitter를 등록한다면 이것은 메모리 누수를 일으키게 됩니다.
  - 애플리케이션에 의해 사용되는 메모리는 무한정 증가할 것이며, 때로는 천천히 때로는 일찍이 그러나 결국에는 애플리케이션을 망가뜨리게 됩니다.
- 이러한 상황을 예방하기 위해서 EventEmitter의 `removeListener()` 메서드로 리스너를 해제할 수 있습니다.

```js
emitter.removeListener('an_event', listener)
```



- EventEmitter는 개발자에게 메모리 누수에 대한 가능성을 경고하기 위해서 매우 간단한 내장 메커니즘을 가지고 있습니다.
- 리스너의 수가 특정 개수 (default 10개)를 초과할 때 EventEmitter는 경고를 발생시킵니다.
- 가끔은 10개 이상의 등록이 전혀 무리 없을 때도 있기에 우리가 EventEmitter의 `setMaxListeners()` 메서드를 사용하여 이것에 대한 제한을 조정할 수 있습니다.

> 첫 번째 이벤트를 받고 나서 자동으로 리스너를 해지하는 편리한 메서드인 `once(event, listener)`를 `on(event, listener)`를 대신해서 사용할 수 있습니다.
>
> 그러나 <u>**발생되지 않는 이벤트를 명시하였을 경우 리스너는 절대 해지되지 않으며**</u> 메모리 누수의 원인이 된다는 점을 상기해야 합니다.



### 3-2-6 동기 및 비동기 이벤트

- 이벤트 또한 콜백과 마찬가지로 이벤트를 생성하는 작업이 호출되는 순간에 따라 동기적 또는 비동기적으로 발생될 수 있습니다.
  - 동일한 EventEmitter에서 두 가지 접근 방식을 섞어서는 안됩니다.
  - 더 중요한 것은 동기와 비동기를 혼합하여 동일한 유형의 이벤트를 발생시키면 안됩니다.
  - 앞서 'Zalgo를 풀어놓다'라는 섹션에서 설명한 것과 같은 문제가 발생하지 않도록 하는 것이 중요합니다.
- 동기와 비동기 이벤트를 발생시키는 주된 차이점은 리스너를 등록할 수 있는 방법에 있습니다.



- 이벤트가 비동기적으로 발생할 때 현재의 스택이 이벤트 루프에 넘어갈 때까지는 이벤트 발생을 만들어내는 작업 이후에도 새로운 리스너를 등록할 수 있습니다.
  - 그 이유는 이벤트가 이벤트 루프의 다음 사이클이 될 때까지 실행되지 않는 것이 보장되기 때문입니다.
  - 따라서 어떠한 이벤트도 놓치지 않게 됩니다.



- 우리가 정의한 `FindRegex` 클래스는 `find()` 메서드가 호출된 이후에 비동기적으로 이벤트를 발생시킵니다.
  - 이것이 이벤트의 손실없이 우리가 `find()` 메서드 호출 이후에도 리스너를 등록할 수 있는 이유입니다.
- 다음의 코드와 같습니다.

```js
findRegexInstance
	.addFile(...)
  .find()
	.on('found', ...)
```



- 반면에 작업이 실행된 이후에 이벤트를 동기적으로 발생시킨다면 모든 리스너를 작업 실행 전에 등록해야 합니다.
  - 그렇지 않으면 모든 이벤트를 잃게 됩니다.
  - 어떻게 작동하는지 살펴보기 위해 우리가 앞서 정의한 `FindRegex` 클래스를 수정하고 `find()` 메서드를 동기적으로 만들어 보겠습니다.

```js
find() {
  for (const file of this.files) {
    let content
    try {
      content = readFileSync(file, 'utf8')
    } catch (err) {
      this.emit('error', err)
    }
    this.emit('fileread', file)
    const match = content.match(this.regex)
    if (match) {
      match.forEach(elem => this.emit('found', file, elem))
    }
  }
  
  return this
}
```

- `find()` 작업을 실행하기 전에 리스너를 등록하겠습니다. 그리고 두 번째 리스너를 작업 이후에 등록해보고 무슨 일이 발생하는지 보도록 하겠습니다.

```js
const findRegexSyncInstance = new FindRegexSync(/hello \w+/)
findRegexSyncInstance
	.addFile('fileA.txt')
	.addFile('fileB.json')
  // 리스너가 호출됨
	.on('found', (file, match) => console.log(`[Before] Matched "${match}"`))
	.find()
	// 이 리스너는 절대 호출안됨
	.on('found', (file, match) => console.log(`[After] Matched "${match}"`))
```

- 예상했던 것처럼 `find()` 작업 이후에 등록한 리스너는 절대 호출되지 않습니다. 앞선 코드의 출력입니다.

```js
[Before] Matched "hello world"
[Before] Matched "hello NodeJS"
```



- 드물게는 동기적 방법에서의 이벤트 발생이 적합할 때가 있습니다.
- 하지만 <u>EventEmitter의 본성은 비동기적 이벤트를 다루는 데에 근거</u>합니다.
  - 이벤트를 동기적으로 발생시키는 것은 우리가 EventEmitter가 필요하지 않거나 어딘가에서 똑같은 관찰 가능한 것이 또 다른 이벤트를 비동기적으로 발생시키고 있다는 신호입니다.
  - 이것은 잠재적으로 Zalgo의 상황입니다.

> 동기적 이벤트 발생은 `process.nextTick()`과 함께 그들이 비동기적으로 발생되는 것을 보장하도록 연기될 수 있습니다.



### 3-2-7 EventEmitter vs 콜백

- 비동기식 API를 정의할 때 공통적인 딜레마는 EventEmitter를 사용할지 아니면 단순하게 콜백을 사용할지를 결정하는 것입니다.
- 일반적인 판단 규칙은 그 의미에 있습니다.
  - 결과가 비동기적으로 반환되어야 하는 경우에는 콜백을 사용하며, 이벤트는 발생한 사건과 연결될 때 사용되어야 합니다.
- 이처럼 간단한 원리에도 두 패러다임 사이에서 많은 혼동이 생겨나는 게 사실입니다. 대부분의 경우 동일하며 같은 결과를 얻게 해줍니다.
- 다음의 예제를 살펴보겠습니다.

```js
import { EventEmitter } from 'events'

function helloEvents() {
  const eventEmitter = new EventEmitter()
  setTimeout(() => eventEmitter.emit('complete', 'hello world'), 100)
  return eventEmitter
}

functino helloCallback(cb) {
  setTimeout(() => cb(null, 'hello world'), 100)
}

helloEvents().on('complete', message => console.log(message))
helloCallback((err, message) => console.log(message))
```

- 두 함수 `helloEvents()`와 `helloCallback()`은 기능적인 면에서 동일하게 생각할 수 있습니다.
- 첫 번째는 이벤트를 사용하여 타임아웃의 완료를 전달하며, 두 번째는 콜백을 사용합니다.
- 그러나 실제로 이들을 구별하는 것은 가독성, 의미, 구현 또는 사용되는데 필요한 코드의 양입니다.



- 어떤 스타일을 선택할지에 대한 규칙을 제공할 수는 없지만 결정을 내리는데 도움이 될 수 있는 몇 가지 힌트는 제공할 수 있습니다.
- **콜백**은 <u>여러 유형의 결과를 전달</u>하는데 있어서 약간의 제한이 있습니다. 
  - 실제로 콜백의 인자로 타입을 전달하거나 각 이벤트에 적합한 여러 개의 콜백을 취하여 차이를 둘 수 있습니다.
  - 하지만 이것은 깔끔한 API라고 할 수 없습니다. 이 상황에서는 EventEmitter가 더 나은 인터페이스와 군더더기 없는 코드를 만들게 해줍니다.
- **EventEmitter**는 <u>같은 이벤트가 여러 번 발생</u>하거나 <u>아예 발생하지 않을 수도 있는 경우</u>에 사용되어야 합니다.
  - 사실 콜백은 작업이 성공적이든 아니든 정확히 한번 호출됩니다.
  - 반복가능성이 있는 상황을 갖는 것은 결과가 반환되어야 하는 것보다는 알려주는 기능인 이벤트와 더 유사합니다.
- **콜백**을 사용하는 API는 오직 특정한 <u>콜백 하나만</u>을 호출할 수 있습니다.
- 반면에, **EventEmitter**는 같은 이벤트에 대해 다수의 리스너를 등록할 수 있게 해줍니다.





### 3-2-8 콜백과 EventEmitter의 결합

- EventEmitter를 콜백과 함께 사용할 수 있는 특정한 상황도 존재합니다.
  - 이 패턴은 매우 강력합니다.
- 전통적인 콜백을 사용하여 결과를 비동기적으로 전달할 수 있게끔 해주고 동시에 EventEmitter를 반환하여 비동기 처리 상태에 대해 보다 상세한 판단을 제공하는데 사용될 수 있습니다.



- 이 패턴의 예시는 glob 스타일로 파일 검색을 수행하는 라이브러리인 `glob` 패키지(https://www.npmjs.com/package/glob)에 의해 제공됩니다.
- 이 모듈의 주요 진입점은 아래와 같은 특징을 가지는 함수입니다.

```js
const eventEmitter = glob(pattern, [options], callback)
```

- 이 함수는 패턴을 첫 번째 인자로 취하고, 다음에는 일련의 옵션을 그리고 주어진 패턴과 일치하는 모든 파일 리스트를 가지고 호출될 콜백 함수를 취합니다.
- 동시에 이 함수는 프로세스 상태에 대해서 보다 세분화된 알림을 제공하는 EventEmitter를 반환합니다.
  - 예를 들어, `match` 이벤트가 일어날 때 실시간으로 알림을 받거나 `end` 이벤트와 함께 매칭되는 모든 파일 리스트를 얻거나 `abort` 이벤트를 받아 프로세스가 수동으로 중단되었는지 아닌지 아는 것이 가능합니다.
- 다음의 코드는 어떻게 사용되는지 보여줍니다.

```js
import glob from 'glob'

glob('data/*.txt', 
  (err, files) => {
  	if (err) {
      return console.error(err)
    }
  	console.log(`All files found: ${JSON.stringify(files)}`)
	})
	.on('match', match => console.log(`Match found: ${match}`))
```

- 전통적인 콜백과 EventEmitter를 결합하는 것은 같은 API에 두 가지 다른 접근을 제공하는 우아한 방법입니다.
- 하나의 접근이 일반적으로 더 간단하고 더욱 즉각적으로 사용되는 것으로 여겨지는 반면, 다른 접근은 심화된 시나리오에서 선택됩니다.



> 또한 EventEmitter는 프라미스("5장. Promises 그리고 Async/Await와 함께 하는 비동기 제어 흐름 패턴"에서 살펴볼 것입니다)와 같이 다른 비동기 메커니즘과 결합될 수도 있습니다.
>
> 이 경우에는 단지 프라미스와 EventEmitter를 모두 포함하는 객체(또는 배열)를 반환합니다.
>
> 이 객체는 호출자에 의해 { promise.events } = foo()와 같이 해체될 수 있습니다.





### 요약

- 이 장에서는 처음으로 실용적 측면의 비동기적 코드 작성을 접해보았습니다.
- 전체 Node.js에서 비동기적 기반의 큰 두 갈래(EventEmitter와 콜백)를 알아보았습니다.
  - 그리고 그것의 사용에 관한 세부사항과 규약 그리고 패턴을 탐구해보았습니다.
- 또한 비동기적 코드를 다룰 때 몇 가지 위험한 것들을 살펴보고 그것을 피하는 방법에 대해서 배웠씁니다.