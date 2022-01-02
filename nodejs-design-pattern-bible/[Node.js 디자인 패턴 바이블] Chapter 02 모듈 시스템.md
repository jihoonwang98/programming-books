# [Node.js 디자인 패턴 바이블] Chapter 02 모듈 시스템



**이번 절에서 배울 것들**

- 모듈이 왜 필수적이며 Node.js에서 다른 모듈시스템이 가능한 이유
- CommonJS 내부와 모듈 패턴
- ES 모듈(ESM)
- CommonJS와 ESM 사이의 차이점 및 상호 이용



**목차**

- 2-1 모듈의 필요성
- 2-2 JavaScript와 Node.js에서의 모듈 시스템
- 2-3 모듈 시스템과 패턴
  - 노출식 모듈 패턴
- 2-4 CommonJS 모듈
  - 직접 만드는 모듈 로더
  - 모듈 정의
  - module.exports 대 exports
  - require 함수는 동기적이다.
  - 해결(resolving) 알고리즘
  - 모듈 캐시
  - 순환 종속성
- 2-5 모듈 정의 패턴
  - exports 지정하기 (Named exports)
  - 함수 내보내기
  - 클래스 내보내기
  - 인스턴스 내보내기
  - 다른 모듈 또는 전역 범위(global scope) 수정
- 2-6 ESM: ECMAScript 모듈
  - Node.js에서 ESM의 사용
  - exports와 imports 지정하기(named exports and imports)
  - export와 import 기본값 설정하기(default exports and imports)
  - 혼합된 export(mixed exports)
  - 모듈 식별자
  - 비동기 임포트
  - 모듈 적재 이해하기
  - 모듈의 수정
- 2-7 ESM과 CommonJS의 차이점과 상호 운용
  - strict 모드에서의 ESM
  - ESM에서의 참조 유실
  - 상호 운용





## 2-1 모듈의 필요성



### 모듈의 필요성

- 코드베이스를 나누어 **여러 파일로 분할**하는 방법을 제시한다.
  - 이것은 코드를 좀 더 구조적으로 관리할 수 있게 해주고 각각으로부터 독립적인 기능의 조각들을 개발 및 테스트하는 데에 도움을 주며 이해하기 쉽게 해준다.
- 다른 프로젝트에 코드를 **재사용**할 수 있게 해준다.
  - 실제로 모듈은 다른 프로젝트에도 유용하고 일반적인 특성을 구현할 수 있다.
  - 모듈로서 기능을 구조화하는 것이 그 기능들이 필요한 다른 프로젝트로 좀 더 쉽게 이동시킬 수 있다.
- **은닉성**을 제공한다.
  - 일반적으로 복잡한 구현을 숨긴 채 명료한 책임을 가진 간단한 인터페이스만 노출시키는 것이 좋은 방식이다.
  - 대부분의 모듈 시스템은 함수와 클래스 또는 객체와 같이 모듈 사용자가 이용하도록 공개 인터페이스를 노출시키는 반면, 공개하지 않으려는 코드를 선택적으로 공개되지 않도록 해준다.
- **종속성**을 관리한다.
  - 좋은 모듈 시스템은 서드파티(third-party)를 포함하여 모듈 개발자로 하여금 기존에 있는 모듈에 의존하여 쉽게 빌드할 수 있게 해준다.
  - 또한 모듈 시스템은 모듈 사용자가 주어진 모듈의 실행에 있어 필요한 일련의 종속성들을 쉽게 임포트할 수 있게 해준다.



- 모듈과 모듈 시스템을 구별하는 것은 중요합니다.
  - **모듈 시스템**이 문법이며 우리의 프로젝트 안에서 모듈을 정의하고 사용할 수 있게 해주는 도구인 반면,
  - **모듈**은 소프트웨어의 실제 유닛으로 정의할 수 있습니다.



## 2-2 JavaScript와 Node.js에서의 모듈 시스템

- 모든 프로그래밍 언어가 내장 모듈 시스템을 가지고 있는 것은 아니며, JS는 오랫동안 이 시스템을 갖고 있지 않았다.
- 하지만 JS 애플리케이션이 점점 복잡해지면서 JS 커뮤니티에는 JS 프로젝트에 효율적으로 사용될 모듈 시스템을 정의하기 위한 여러 시도가 나타나기 시작했다.
- 가장 성공적인 것이 **AMD(Asynchronous Module Definition)**이다.
  - AMD는 RequireJS에 의해서 대중화되었고 후에는 **UMD(Universal Module Definition)**가 나오게 된다.



- Node.js가 처음 만들어졌을 때, Node.js는 OS의 파일시스템에 직접적으로 접근하는 JS를 위한 서버 런타임으로서 구상되었다.
  - 그래서 모듈 관리에 있어서 다른 방법을 도입할 수 있는 특별한 기회가 있었다.
  - 이때 도입된 방법이란 HTML `<script>`과 URL을 통한 리소스 접근에 의존하지 않고 <u>오직 로컬 파일시스템의 JS 파일들에만 의존하는 것</u>이었다.
  - 이 모듈 시스템을 도입하기 위해 Node.js는 브라우저가 아닌 환경에서 JS 모듈 시스템을 제공할 수 있도록 고안된 **CommonJS**(종종 CJS라 불림)의 명세를 구현하게 되었다.
- CommonJS는 Node.js에서 주된 모듈 시스템이 되었고 Browserfiy와 Webpack과 같은 모듈 번들러 덕분에 브라우저 환경에서도 유명세를 가지게 되었다.



- 2015년에 ECMAScript 6(ECMAScript 2015 또는 ES2015로 불리기도 하는)의 발표와 함께 표준 모듈 시스템 (ESM 또는 ECMAScript Modules)을 위한 공식적인 제안이 나오게 된다.
  - ESM은 JavaScript 생태계에 많은 혁신을 불러왔다.
  - 특히, 모듈 관리에 대한 브라우저와 서버의 차이점을 연결하기 위해 노력했다.
- ECMAScript 6는 문법과 의미론적 관점에서 ESM을 위한 공식적인 명세만을 정의하고 구체적인 구현을 제공하지 않았다.
  - 다른 여러 브라우저 회사들과 Node.js 커뮤니티가 확실한 명세를 구현하는 데 몇 년이 소요되었다.
  - Node.js는 버전 13.2부터 ESM에 대한 안정적인 지원을 하고 있다.
- 이 글이 작성되는 시점에는 브라우저와 서버 환경 모두에서 ESM이 JS 모듈을 관리하는 실질적인 방법이 되어 가고 있다.
- 그러나 지금 시점에는 대부분의 프로젝트들이 CommonJS에 절대적으로 의존하고 있으며 ESM이 지배적인 표준이 되는 데에는 어느 정도의 시간이 소요될 것이다.



- Node.js에서 모듈 관련 사항에 대한 포괄적인 개요를 제공하기 위해서 이번 장의 첫 번째 부분에서는 CommonJS의 관점에서 이야기해 볼 것이며, 두 번째 부분에서는 ESM을 사용해 설명을 진행한다.





## 2-3 모듈 시스템과 패턴

- 앞서 언급했듯이 **모듈**은 주요 애플리케이션의 구조화를 위한 부품인 동시에 명시적으로 노출시키지 않은 모든 함수들과 변수들을 비공개로 유지하여 정보에 대한 은닉성을 강화시켜주는 주된 장치이다.
- CommonJS를 구체적으로 알아보기 전에 우리가 간단한 모듈 시스템으로 만들어보기 위해 사용할 패턴이자 정보를 감추는 데에 도움을 주는 **노출식 모듈 패턴**을 이야기해보자.



### 2-3-1 노출식 모듈 패턴

- JS의 주요 문제점 중 하나는 네임스페이스가 없다는 것이다.
  - 모든 스크립트는 전역 범위에서 실행된다.
  - 따라서 내부 애플리케이션 코드나 종속성 라이브러리가 그들의 기능을 노출시키는 동시에 스코프를 오염시킬 수 있다.
  - 예를 들어, 종속성 라이브러리가 전역 변수 `utils`를 선언했다고 생각해보자.
    - 만약 다른 라이브러리나 애플리케이션 코드가 의도치 않게 `utils`를 덮어쓰거나 대체해버린다면 그것에 의존하던 코드는 예측 불가능한 상황 속에서 충돌이 일어날 것이다.
  - 다른 라이브러리나 애플리케이션 코드가 내부적으로 사용할 의도를 갖고 함수를 호출할 때에도 예측 불가능한 부작용이 발생할 수 있다.
  - 다시 말해, 전역 범위에 의존하는 것은 매우 위험한 작업이다.
  - 게다가, 애플리케이션이 확장됨에 따라 더욱 개별적인 기능 구현에 의존해야 하는 상황이 발생한다.



- 이러한 문제를 해결하기 위한 보편적인 기법을 **노출식 모듈 패턴(revealing module pattern)**이라고 하며, 다음과 같은 형식을 보인다.

```js
const myModule = (() => {
  const privateFoo = () => { };
  const privateBar = [];
  
  const exported = {
    publicFoo: () => { },
    publicBar: () => { }
  }
  
  return exported;
})()  // 여기서 괄호가 파싱되면, 함수는 호출된다.

console.log(myModule)
console.log(myModule.privateFoo, myModule.privateBar)

[출력 결과]
{ publicFoo: [Function: publicFoo], publicBar: [Function: publicBar] }
undefined undefined
```

- 이 패턴은 자기 호출 함수를 사용한다.
- 이러한 종류의 함수를 **즉시 실행 함수 표현식(IIFE: Immediately Invoked Function Expression)**이라고 부르며, private 범위를 만들고 공개될 부분만 내보내게 된다.



- JS에서는 함수 내부에 선언한 변수는 외부 범위에서 접근할 수 없다.
  - 함수는 선택적으로 외부 범위에 정보를 전파시키기 위해서 `return` 구문을 사용할 수 있다.
- 이 패턴은 비공개 정보의 은닉을 유지하고 공개될 API를 내보내기 위해서 이러한 특성을 핵심적으로 잘 활용하였다.
- 앞선 코드에서 `myModule` 변수는 exported된 API만 포함하고 있으며, 모듈 내용의 나머지 부분은 사실상 외부에서 접근이 불가능하다.
- 이러한 패턴을 기반으로 하는 아이디어가 CommonJS 모듈 시스템에서도 사용된다.





## 2-4 CommonJS 모듈

- CommonJS는 Node.js의 첫 번째 내장 모듈 시스템이다.
- Node.js의 CommonJS는 CommonJS 명세를 고려하여 추가적인 자체 확장 기능과 함께 구현되었다.
- CommonJS 명세의 두 가지 주요 개념을 요약하면 다음과 같다.
  - require는 로컬 파일 시스템으로부터 모듈을 임포트하게 해준다.
  - exports와 module.exports는 특별한 변수로서 현재 모듈에서 공개될 기능들을 내보내기 위해서 사용된다.



### 2-4-1 직접 만드는 모듈 로더

- Node.js에서 CommonJS가 어떻게 작동하는지 이해하기 위해서 비슷한 시스템을 만들어 보겠다.
  - 다음의 코드는 Node.js의 require() 함수의 원래 기능 중 일부를 모방한 함수를 만든 것이다.



- 먼저 모듈의 내용을 로드하고 이를 private 범위로 감싸 평가하는 함수를 작성해보자.

```js
function loadModule(filename, module, require) {
  const wrappedSrc = 
        `(function (module, exports, require) {
        	${fs.readFileSync(filename, 'utf8')}
        })(module, module.exports, require)`;
  
  eval(wrappedSrc);
}
```

- 모듈의 소스코드는 노출식 모듈 패턴과 마찬가지로 기본적으로 함수로 감싸진다.
- 여기서 차이점은 일련의 변수들 (`module`, `exports` 그리고 `require`)을 모듈에 전달한다는 것이다.
- 눈여겨봐야 할 점은 래핑 함수의 `exports` 인자가 `module.exports`의 내용으로 초기화되었다는 것이다.
- 또 다른 주요 사항은 모듈의 내용을 읽어들이기 위해 `readFileSync`를 사용했다는 것이다.
  - 파일시스템의 동기식 버전을 사용하는 것은 일반적으로 권장되지 않지만 여기서는 이것의 사용이 적절하다.
  - CommonJS에서 모듈을 로드하는 것이 의도적인 동기 방식이기 때문이다.
  - 이러한 방식에서는 여러 모듈을 임포트할 때 올바른 순서를 지키는 것이 중요하다.



- 지금부터 `require()` 함수를 구현해보자.

```js
function require(moduleName) {
  console.log(`Require invoked for module: ${moduleName}`);
  const id = require.resolve(moduleName);   // (1)
  
  if(require.cache[id]) {   // (2)
    return require.cache[id].exports;
  }
  
  // 모듈 메타데이터
  const module = {    // (3)
    exports: {},
    id
  }
  
  // 캐시 업데이트
  require.cache[id] = module  // (4)
  
  // 모듈 로드
  loadModule(id, module, require)  // (5)
  
  // 익스포트되는 변수 반환
  return module.exports  // (6)
}

require.cache = {}
require.resolve = (moduleName) => {
  /* 모듈이름으로 id로 불리게 되는 모듈의 전체 경로를 찾아냄 (resolve) */
}
```

- 위의 함수는 Node.js에서 모듈을 로드하기 위해 사용되는 Node.js `require()` 함수의 동작을 모방하고 있다.

- 물론 이는 교육적인 목적을 위한 것이며, 실제 `require()` 함수의 내부 동작을 정확하고 완전하게 반영하고 있지 않다.

  - 그러나 모듈이 어떻게 정의되고 로드되는지를 포함해서 Node.js 모듈 시스템의 내부를 이해하기에는 부족함이 없을 것이다.

- 우리가 작성한 모듈 시스템은 다음과 같이 설명된다.

  1. 모듈 이름을 입력으로 받아 수행하는 첫 번째 일은 우리가 id라고 부르는 모듈의 전체 경로를 알아내는(resolve) 것이다.

     이 작업은 이를 해결하기 위해 관련 알고리즘을 구현하고 있는 `require.resolve()`에 위임된다.

  2. 모듈이 이미 로드된 경우 캐시된 모듈을 사용한다. 이 경우 즉시 반환된다.

  3. 모듈이 아직 로드되지 않은 경우 최초 로드를 위해서 환경을 설정한다. 특히 빈 객체 리터럴을 통해 초기화된 `exports` 속성을 가지는 `module` 객체를 만든다. 

     이 객체는 불러올 모듈의 코드에서의 public API를 export 하는데 사용된다.

  4. 최초 로드 후에 `module` 객체가 캐시된다.

  5. 모듈 소스코드는 해당 파일에서 읽어오며, 코드는 앞에서 살펴본 방식으로 평가된다. 방금 생성한 `module` 객체와 `require()` 함수의 참조를 모듈에 전달한다.

     모듈은 `module.exports` 객체를 조작하거나 대체하여 public API를 내보낸다.

  6. 마지막으로, 모듈의 public API를 나타내는 `module.exports`의 내용이 호출자에게 반환된다.

- 우리가 본 바와 같이, Node.js 모듈 시스템은 마법이 아니다. 트릭이라면 모듈의 소스코드를 둘러산 래퍼(wrapper)와 실행을 위해 인위적으로 조정한 실행 환경 정도가 전부다.



### 2-4-2 모듈 정의

- 우리가 만든 `require()` 함수가 어떻게 작동하는지 살펴봄으로써, 모듈을 어떻게 정의하는지 이해할 수 있게 되었다.
- 다음의 코드는 그 예를 보여준다.

```js
// 또 다른 종속성 코드
const dependency = require('./anotherModule');

// private 함수
function log() {
  console.log(`Well done ${dependency.username}`);
}

// 공개적으로 사용되기 위해 익스포트되는 API
module.exports.run = () => {
  log();
}
```

- 기억해야 할 기본 개념은 `module.exports` 변수에 할당되지 않는 이상, 모듈 안의 모든 것이 비공개라는 것이다.
- `require()`를 사용하여 모듈을 로드할 때 변수의 내용은 캐시되고 리턴된다.

















