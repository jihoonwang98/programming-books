# [Programming TypeScript] 10. Namespaces.Modules

**이번 절에서 다루는 것들**

- A Brief History of JavaScript Modules
- import, export
  - Dynamic imports
  - Using CommonJS and AMD Code
  - Module Mode Versus Script Mode
- Namespaces
  - Collisions
  - Compiled Output
- Declaration Merging



## \#1. A Brief History of JavaScript Modules

- TS는 JS를 컴파일하고 JS와 상호작용하기 때문에, JS 프로그래머들이 사용하는 다양한 module standard들을 지원해야 한다.

- JS 모듈의 역사에 대해 알아보자.

- 1995년 초창기에 JS는 module system을 지원하지 않았다.

- module 없이 모든 것은 global namespace에 선언되었다.

- 모듈과 비슷한 기능을 만들어내기 위해 사람들은 객체나 Immediately Invoked Function Expressions(IIFEs, 즉시 실행 함수)를 이용했다.

- 이하 생략

  

## \#2. import, export

- 특별한 이유가 없다면, CommonJS, global, namespaced modules보다는 ES2015의 `imports`와 `exports`를 사용하자.

```ts
// a.ts
export function foo() {}
export function bar() {}

// b.ts
import {foo, bar} from './a'
foo()
export let result = bar()
```



- default exports

```ts
// c.ts
export default function meow(loudness: number) {}

// d.ts
import meow from './c' 
meow(11)
```



- wildcard import(*)

```ts
// e.ts
import * as a from './a';
a.foo()
a.bar()
```



- reexporting some(or all) export from a module

```ts
// f.ts
export * from './a'
export {result} from './b'
export meow from './c'
```



- 우리는 TypeScript를 사용하고 있기 때문에, 값뿐만 아니라 types와 interfaces들도 export할 수 있다.
- 



## \#3. Namespaces

## \#4. Declaration Merging













