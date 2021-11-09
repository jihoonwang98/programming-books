# [Programming TypeScript] 3. All About Types

**이번 절에서 다루는 것들**

- Talking about Types
- The ABCs of Types
  - any
  - unknown
  - boolean
  - number
  - bigint
  - string
  - symbol
  - Objects
  - Intermission: Type Aliases, Unions, and Intersections
  - Arrays
  - Typles
  - null, undefined, void and never
  - Enums
- Summary



## Introduction

- Type의 정의
  - A set of values and the things you can do with them
  - 어떤 값들과 그 값들에 행할 수 있는 연산들을 모아놓은 집합
- 위의 정의가 와닿지 않는다면, 아래의 예시를살펴보자.
  - `boolean` 타입
    - 모든 boolean 값들(`true`, `false`)과 이 boolean 값들에 수행할 수 있는 연산들(`||`, `&&`, `!`)을 모아놓은 하나의 집합이다.
- 즉, 어떤 것이 타입 `T`임을 알았을 때, 해당 값이 `T`임을 알 수 있을 뿐만 아니라, 해당 `T`값을 통해 어떤 연산을 할 수 있는지도 알 수 있다.
- 기억하라, 당신이 유효하지 않은 행동을 하지 않게 막아주는 것은 typecheker를 사용하는 것뿐이다.
  - 그리고 typecheker는 '무엇이 valid한 행동인지'를 당신이 사용하고 있는 타입을 분석하고 그 타입들을 어떻게 사용하고 있는지를 통해서 판가름한다. 

![image-20211108091916983](/Users/mojo/Library/Application Support/typora-user-images/image-20211108091916983.png)



## Talking about Types

- 예시

  ```ts
  function squareOf(n) {
  	return n * n;
  }
  
  squareOf(2);
  squareOf('z')  // evaluates to NaN
  ```

  - 위 함수는 number 타입에 대해서만 작동한다.
    - 문자열을 넘겨주면 결과 값이 유효하지 않다.
  - 따라서, 우리는 명시적으로 파라미터의 타입을 annotate(명시)해줘야 한다.

  ```ts
  function squareOf(n: number) {
    return n * n;
  }
  squareOf(2);
  squareOf('z');  // Error TS2345: Argument of type '"z"' is not assignable to parameter of type 'number'.
  ```

  - 이제 숫자가 아닌 값을 넘겨주면 TS가 complain을 쏟아내기 시작한다.

  

## The ABCs of Types



### \# `any` 타입

- `any` 타입은 모든 타입들의 조상이다.
  - 하지만 선택의 여지가 없는 경우가 아니라면 `any`를 쓰는 것은 좋지 않다.
- TS에서는 컴파일 시점에 모든 것이 type을 가져야만 한다.
  - 따라서, 개발자(당신)이나 TS(typechecker)가 어떤 것이 어떤 타입인지 규명할 수 없을 때는 `any`를 기본값으로 사용하게 된다.
  - 즉, `any` 타입은 최후의 수단인 것이다.
  - 되도록이면 `any` 타입을 사용하는 것을 피해야 한다.
- TS가 어떤 값이 `any` 타입을 가진다고 추론하는 경우(함수의 파라미터에 타입 명시를 안한경우, 타입 명시가 안되어 있는 JS 모듈을 import한 경우 등) TS는 컴파일-타임 예외를 던지고 에디터에서 빨간줄을 그어버린다.
  - `any` 타입을 사용하려면 명시적으로 `: any`라고 써줘야 예외를 피할 수 있다.

- **TSC Flag: noImplicitAny**
  - 기본적으로, TS는 관대해서 `any`로 추론되는 값들에 대해 complain을 늘어놓지 않는다.
  - TS에게 암시적인 `any`값들에 대해 complain하기를 원한다면, `tsconfig.json` 파일에 `noImplicitAny` 옵션을 켜놓아야 한다.
  - `noImplicitAny`는 `strict`를 켜놓으면 자동으로 켜진다.



### \# `unknown` 타입

- `any`가 대부(Godfather)라면, `unknown`은 잠복 FBI 요원입니다. 

  - 느긋하고 나쁜 사람과 잘 어울리지만 깊은 내면에는 법을 존중하고 좋은 사람 편입니다.

- **<u>type을 미리 알지 못하는 값</u>**들을 사용하게 되는 경우에는 `any`를 사용하지 말고 `unknown`을 사용하세요.

- `unknown`은 `any`처럼 모든 값을 나타낼 수 있지만, Ts는 알지 못하는 타입을 확인하여 구체화할 때까지 알 수 없는 타입을 사용하는 것을 허용하지 않습니다.

- `unknown` 값들로 할 수 있는 연산은 무엇이 있을까?

  - `unknown` 값들간 비교 (`==`, `===`, `||`, `&&`, `?`)
  - negate(부정) (`!`)
  - refine with JS's `typeof`, `instanceof`

- `unknown`을 다음과 같이 사용하라:

  ```ts
  let a: unknown = 30;
  let b = a === 123;
  let c = a + 10;
  if(typeof a === 'number') {
    let d = a + 10;
  }
  ```

  - TS는 절대 어떤 것을 `unknown`으로 추론하지 않습니다 - `unknown`을 쓰려면 명시적으로 표기해야 합니다. (변수 `a` 처럼)
  - 당신은 `unknown` 타입의 값을 비교할 수 있습니다. (`b`)
  - 하지만, `unknown` 값이 특정 타입에 속한다고 가정하고 어떤 연산을 할 수는 없다. (`c`)
    - 해당 값이 실제 그 타입에 속하는 값임을 먼저 TS에게 증명해줘야 한다. (`d`)





### \# `boolean` 타입

- Type literal이란?
  - A type that represents a single value and nothing else.



### \# `symbol` 타입

- unique symbol
  - 새로운 `symbol`을 선언하고 `const` 변수에 할당할 때, TS는 해당 값의 타입을 `unique symbol`로 추론한다.
    - code editor에서는 `unique symbol`이 아니라 `typeof [yourVariableName]`으로 나타날 것이다.
  - `const` 변수의 타입을 명시적으로 `unique symbol`로 명시할 수 있다.
  - `unique symbol`은 언제나 자신과 equal하다.
  - TS는 컴파일 시점에 어떤  `unique symbol`은 다른 `unique symbol`들과 절대로 같을(equal) 수 없음을 알고 있다.





### \# `Objects`

- TS의 object type은 객체의 shape을 지정한다.
- TS는 simple object(`{}`)와 더 복잡한 객체(`new 머시기()`)를 구분하지 못한다.
  - 이것은 설계에 의한 것이다.
  - JS가 generally structurally하게 타이핑하므로, TS도 anominally typed style보다 JS의 방식을 선호한다.
- Structural typing이란?
  - 어떤 객체가 특정 프로퍼티가 있는지에만 신경쓰고, 그 프로퍼티의 이름이 무엇인지에는 신경쓰지 않는 프로그래밍 스타일
  - duck typing이라고도 불린다.

> [Structural vs Nominal Typing](https://medium.com/@thejameskyle/type-systems-structural-vs-nominal-typing-explained-56511dd969f4)



- TS에서 객체를 묘사할 때 type을 사용하는 몇가지 방법들이 존재한다.

  - 첫번째가 `object`로서 value를 선언하는 것이다.

    ```ts
    let a: object = {
    	b: 'x'
    }
    ```

    - `b`에 접근하면 어떻게 될까?

    ```ts
    a.b // Error TS2339: Property 'b' does not exist on type 'object'.
    ```

    - 엥?? 에러가 난다. 그러면 뭐하러 `object`로 타입을 지정하는 걸까? 아무것도 못하는데...

    - 사실, `object`는 `any`보다는 아주 쪼금 더 구체적인 타입이지만, 그렇다고 엄청 구체적인 타입은 아닙니다.

    - `object`는 해당 값이 묘사하는 것에 대해 많은 것을 알려주지 않습니다.

    - `object`라는 것은 그저 해당 값이 JS 객체라는 것(그리고 그것이 `null`이 아니라는 것)을 알려줄 뿐입니다.

      

- 그럼 explicit annotaion을 끄고, TS에게 추론시키면 어떨까?

  ```ts
  let a = {  
    b: 'x'
  }            // {b: string}
  a.b          // string
  
  let b = {  
    c: {
      d: 'f'  
    }
  }            // {c: {d: string}}
  ```

  - You’ve just discovered the second way to type an object: object literal syntax(not to be confused with type literals).

- You can either let TS infer your object's shape for you, or explicitly describe it inside curly braces(`{}`)

  ```ts
  let a: {b: number} = { 
  	b: 12
  }
  ```

  

- Type Inference when declaring objects with const

  - What would have happend if we'd used `const` to declare the object instead?

    ```ts
    const a: {b: number} = {
    	b: 12
    }  // Still { b: number }
    ```

    - 당신은 TS가 `b`를 literal `12`가 아니라 `number`로 추론한 것에 대해 놀랐을 수도 있다.

  - 결국 우리는 `numbers`나 `strings`들을 선언할 때 `const`를 쓰냐 `let`을 쓰냐에 따라 TS가 type을 추론하는데 영향을 미침을 배웠다.

- primitive type과는 다르게 `object`를 `const`로 선언하는 것은 TS에게 해당 타입을 좀 더 좁게(구체적으로) 추론하는데 도움을 주진 않는다.

  - 이는 JS 객체들이 mutable하기 때문이다. (`const`로 선언한 객체의 프로퍼티를 변경하는 것은 가능하다.)



- Object literal syntax는 TS에게 "이런 모양을 가진 것(a thing)가 존재해!!"라고 말해준다.

  - the thing은 object literal이 될 수도 있고 class가 될 수도 있다.

  ```ts
  let c: {
    firstName: string;
    lastName: string;
  } = {
    firstName: 'john',
    lastName: 'barrowman'
  };
  
  class Person {
    constructor(
      public firstName: string,
      public lastName: string
    ) {
    }
  }
  
  c = new Person('matt', 'smith');
  ```

  - `{firstName: string; lastName: string}`은 객체의 shape을 묘사한다.
  - 그리고 object literal과 class instance는 모두 위의 shape을 만족시킬 수 있으므로 TS는 `Person`을 `c`에 할당하게 해준다.





- 추가 프로퍼티를 같이 넘겨주는 경우 또는 required 프로퍼티를 안넘겨주는 경우

  ```ts
  let a: {b: number};
  a = {};  // 명시된 required property(b)를 안넘겨주는 경우
           // Error TS2741: Property 'b' is missing in type '{}'
  
  a = {
    b: 1,
    c: 2   // Error TS2322: Type '{b: number, c: number}' is not assignable
  }        // to type '{b: number}'.  'c' does not exist in type '{b: number}'.
   
  ```

  

- 기본적으로 TS는 object properties에 관해서는 매우 strict하다 - 만약 어떤 객체가 `number` 타입의 `b`라고 불리는 프로퍼티가 존재해야 한다고 하면, TS expects `b` and only `b`.
- 만약 `b` 가 없거나 다른 extra property가 존재하면 TS는 complain을 뿜는다.



- TS에게 어떤 프로퍼티가 optional하거나 다른 프로퍼티가 추가적으로 들어올 수도 있다는 것을 어떻게 알릴 수 있을까?

  ```ts
  let a: { 
  	b: number  (1)
    c?: string  (2)
    [key: number]: boolean  (3)
  }
  ```

  - (1): `a`는 숫자 타입의  `b`라는 프로퍼티를 가지고 있다.
  - (2): `a`는 string 타입의 `c`라는 프로퍼티를 가질 수 있다. `c`가 set 되었을 때, `undefined`일 수 있다.
  - (3): `a`는 any number of numeric properties that are booleans를 가질 수 있다.

- `a`에 할당할 수 있는 객체에 어떤 종류가 있는지 살펴보자.

  ```ts
  a = {b: 1}
  a = {b: 1, c: undefined}
  a = {b: 1, c: 'd'}
  a = {b: 1, 10: true}
  a = {b: 1, 10: true, 20: false}
  a = {10: true}  // Error TS2741: Property 'b' is missing in type
  a = {b: 1, 33: 'red'}  // Error TS2741: Type 'string' is not assignable to type 'boolean'.
  ```



- 인덱스 시그니처 (Index Signatures)

  - `[key: T]: U` 문법은 index signature라고 불리는 문법니다.
    - 이를 통해 TS에게 해당 객체가 더 많은 key를 가질 수 있음을 알려줄 수 있다.
    - 이 문법을 읽는 방법은, "이 객체에 대해서 type `T`를 가지는 모든 key들은 type `U`를 가지는 value들을 가져야만 한다."
  - Index signature는 당신이 명시적으로 선언한 key들 외에도 객체에 더 많은 key들을 안전하게 추가할 수 있게 해준다.
  - 이때, index signature에 대해 명심해야 할 rule이 하나 있다.
    - index signature key의 type `T`는 `number` 또는 `string`에 assignable 해야 한다.
  - 그리고 당신은 index signature key의 이름으로 아무 글자나 쓸 수 있다. 

  



- Optional(`?`)은 객체 타입을 선언할 때 사용할 수 있는 유일한 modifier가 아닙니다.
- 당신은 필드에 대해 read-only로도 mark할 수 있습니다. `readonly` 수정자를 통해...
  - 즉, 당신은 어떤 필드가 초기값이 할당된 이후에 수정될 수 없음을 선언할 수 잇습니다.
  - 마치 객체 프로퍼티의 `const` 처럼..

```ts
let user: {
	readonly firstName: string
} = {
  firstName: 'abby'
}

user.firstName // string
user.firstName = 'abbey with an e'  // Error TS2540: Cannot assign to 'firstName' because it is a read-only property
```



- Object literal notation은 한가지 special case가 잇습니다: empty object types(`{}`)
- `null`과 `undefined`를 제외한 모든 type은 empty object type에 할당 가능하다.
  - 이런 사실은 사용하기 헷갈리게 만들 수 잇으므로 되도록이면 empty object type 사용을 가능한 피하라.

```ts
let danger: {}
danger = {};
danger = {x: 1};
danger = [];
danger = 2;
```





- 요약 / TS에서 objects를 선언하는 4가지 방법
  1. **Object literal notation (like `{a: string}`), also called a shape.**
     - 이 방법은 당신이 당신의 객체에 들어갈 수 잇는 필드에 대해 알거나 당신의 객체의 값들이 모두 같은 type을 가져야 할 때 사용하라.
  2. Empty object literal notation(`{}`)
     - 가능한 사용을 피하라.
  3. **`object` 타입**
     - 당신이 그저 object를 원하고 그 안에 어떤 필드가 들어있을지에 대해서는 관심이 없을 때 사용하라.
  4. `Object`  타입
     - 가능한 사용을 피하라.



**[table 3-1] Is the value a valid object?**

| Value           | {}   | object | Object |
| --------------- | ---- | ------ | ------ |
| {}              | Yes  | Yes    | Yes    |
| ['a']           | Yes  | Yes    | Yes    |
| function () {}  | Yes  | Yes    | Yes    |
| new String('a') | Yes  | Yes    | Yes    |
| 'a'             | Yes  | No     | Yes    |
| 1               | Yes  | No     | Yes    |
| Symbol('a')     | Yes  | No     | Yes    |
| null            | No   | No     | No     |
| undefined       | No   | No     | No     |







### Intermission: Type Aliases, Unions, and Intersections

- 당신이 어떤 값의 type에 대해 알고 있으면, 그 값에 어떤 연산을 수행할 수 있는지도 알 수 있다.





#### Type aliases

- 당신이 어떤 값을 aliases하는 variable을 선언하기 위해 변수 선언(`let`, `const`, `var`)을 사용하는 것처럼,

  어떤 타입을 가리키는 type alias를 선언할 수 있다.

  ```ts
  type Age = number
  
  type Person = {
    name: string;
    age: Age;
  }
  ```

- 위와 같이 Type alias를 사용하면 타입을 더 이해하기 쉬워진다.

- Aliases들은 TS에 의해 절대 추론되지 않으므로 당신은 얘네들을 명시적으로 type 해줘야 한다.

  ```ts
  let age: Age = 55;
  
  let driver: Person = {
  	name: 'James May',
    age: age
  }
  ```



- `let`과 `const`처럼 type aliases들은 block-scope를 갖는다.

  - Every block and every function has its own scope, and inner type alias declarations shadow outer ones

    ```ts
    type Color = 'red';
    
    let x = Math.random() < .5
    
    if (x) {
      type Color = 'blue'; // This shadows the Color declared above.
      let b: Color = 'blue'
    } else {
      let c: Color = 'red';
    }
    ```



#### Union and Intersection Types

- TS는 type들에 대한 union과 intersection을 묘사하기 위해 special type operator들 (`|`, `&`)을 지원한다.

  ```ts
  type Cat = {name: string, purrs: boolean}
  type Dog = {name: string, barks: boolean, wags; boolean}
  type CatOrDogOrBoth = Cat | Dog
  type CatAndDog = Cat & Dog
  ```

- 만약 어떤 것이 `CatOrDogOrBoth`라는 게 의미하는게 뭔가?

  - 당신은 그것이 string 타입의 `name`을 갖고 있다는 것만을 알 수 있다.

- 반대로, `CatOrDogOrBoth`에 무엇을 할당할 수 있는가?

  - `Cat`, `Dog`, 둘 다도 가능하다.

  ```ts
  // Cat
  let a: CatOrDogOrBoth = {
  	name: 'Bonkers',
    purrs: true
  }
  
  // Dog
  a = {
    name: 'Domino',
    barks: true,
    wags: true
  }
  
  // Both
  a = {
    name: 'Donkers',
    barks: true,
    purrs: true,
    wags: true
  }
  ```

  - a value with a union type (`|`)은 union의 특정한 member일 필요가 없다. 
  - 동시에 두 멤버일 수 있다.

- On the other hand, `CatAndDog`에 대해서는 무엇을 알 수 있을까?

  - 이 잡종 슈퍼 개-고양이 애완동물은 이름이 있을 뿐만 아니라 purr, bark, wag도 가능하다.



- 자연스럽게도 intersection보다는 union을 더 많이 사용하게 된다.

  ```ts
  function trueOrNull(isTrue: boolean) {
  	if(isTrue) {
      return 'true'
    }	
    return null
  }
  ```

  - 이 함수가 반환하는 값의 타입은 무엇인가?
    - `string` 또는 `null`일 것이다.
    - 이를 다시 나타내면, `type Returns = string | null`로 나타낼 수 있다.

  

  ```ts
  function(a: string, b: number) {
  	return a || b
  }
  ```

  - 얘도 `string | number` 형태로 리턴



### Arrays

```ts
let a = [1, 2, 3]  // number[]
let c: string[] = ['a'] // string[]
const e = [1, 'a']  // (string | number)[]

let f = ['red']
f.push('blue')
f.push(true)  // Error TS2345: Argument of type 'true' is not assignable to parameter of type 'string'.

let g = []  // any[]
g.push(1)  // number[]
g.push('red')  // (string | number)[]


let h: number[] = [];
h.push(1)  // number[]
h.push('red')  // Error TS2345: Argument of type '"red"' is not assignable to parameter of type 'number'.
```



- TS는 array를 위해 다음 두가지 syntax를 지원한다. 
  - `T[]`
  - `Array<T>`
- 얘네들은 아예 identical한 놈이다.





### Tuples















