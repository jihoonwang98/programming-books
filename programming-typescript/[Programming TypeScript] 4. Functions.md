# [Programming TypeScript] 4. Functions

**이번 절에서 다루는 것들**

- Declaring and Invoking Functions
  - Optional and Default Parameters
  - Rest Parameters
  - call, apply, and bind
  - Typing this
  - Generator Functions
  - Iterators
  - Call Signatures
  - Contextual Typing
  - Overloaded Function Types
- Polymorphism
  - When Are Generics Bound?
  - Where Can You Declare Generics?
  - Generic Type Inference
  - Generic Type Aliases
  - Bounded Polymorphism
  - Generic Type Defaults
- Type-Driven Development
- Summary



## Declare and Invoking Functions



### Optional and Default Parameters

#### \# Optional Parameter (`?`)

- 파라미터 순서

  - Required Parameter(필수 파라미터) -> Optional Parameter가 와야함.
  - Optional parameter들은 맨 뒤에 와야함.

  ```ts
  function log(message: string, userId?: string) {
  	let time = new Date().toLocalTimeString();
    console.log(time, message, userId || 'Not signed in');
  }
  
  log('Page loaded'); // "12:38:31 PM Page loaded Not Signed in"
  log('User signed in', 'da763be'); // "12:38:31 PM User Signed in da763be"
  ```

  

#### \# Default Parameter (`=`)

- 파라미터 순서

  - default parameter들은 꼭 맨 뒤에 와야 할 필요가 없다.

  ```ts
  function log(message: string, userId = 'Not signed in') {
  	let time = new Date().toLocalTimeString();
    console.log(time, message, userId);
  }
  
  log('Page loaded'); // "12:38:31 PM Page loaded Not Signed in"
  log('User signed in', 'da763be'); // "12:38:31 PM User Signed in da763be"
  ```



### Rest Parameters

- JS의 `arguments` 

  - 만약 함수가 argument를 여러개 받아야 할때, 배열로 받을 수 있다.

    ```ts
    function sum(numbers: number[]): number {
      return numbers.reduce((total, n) => total + n, 0)
    }
    sum([1, 2, 3]) // evaluates to 6
    ```

  - 하지만 가끔은 가변(variadic) 함수 API를 선택할 수 있다.

    - 가변 함수 API는 정해지지 않은 개수의 argument를 받는다.
    - 이와 반대로 fixed-arity API는 고정된 개수의 argument를 받는다.

  - 이렇게 variadic function API를 사용하려면 JS의 마법같은 `arguments` 객체를 사용해야 한다.

  - `arguments` 객체가 마법이라 불리는 이유는 JS 런타임이 자동으로 이 `arguments` 객체를 함수 내에 정의해주고, 해당 함수에 당신이 넘겨준 arguments 리스트를 이 객체에 할당해주기 때문이다.

  - 하지만 `arguments` 객체는 유사-배열 객체이지, 진짜 배열은 아니므로 `.reduce` 메서드를 호출하기 위해서는 먼저 배열로 변환시켜줘야 한다.

    ```ts
    function sumVariadic(): number {
      return Array
              .from(arguments)
              .reduce((total, n) => total + n, 0);
    }
    sumVariadic(1, 2, 3) // 더 이상 배열로 넘겨주지 않는다.
    ```

- 그렇다면 이렇게 `arguments` 객체를 이용하는 것은 괜찮은 방법일까?

  - **<u>그렇지 않다!! 위험하다!!</u>**

  ![image-20211107162256638](/Users/mojo/Library/Application Support/typora-user-images/image-20211107162256638.png)

  - 위 그림을 보면 알 수 있듯이, TypeScript는 reduce의 parameter `total`과 `n`의 타입을 `any`로 추론할 수 밖에 없게 된다.
    - 뭐가 넘어올지 모르니깐..
  - 그리고 사실 컴파일도 안된다.

  ```ts
  sumVariadic(1, 2, 3) // Error TS2554: Expected 0 arguments, but got 3
  ```

  - `sumVariadic` 함수가 아무 인자도 받지 않게 선언했으므로, TypeScript가 보기에는 인자가 넘어오면 잘못되었다고 생각하게 된다.
    - 따라서 이 함수에 인자를 넘겨줘서 사용하게 되면 `TypeError`를 만나게 된다.

- 그렇다면 어떻게 안전한 방법으로 type variadic function을 사용할 수 있을까?

  - **<u>Rest parameter를 사용하자!!</u>**

  ```ts
  function sumVariadicSafe(...numbers: number[]): number {
    return numbers.reduce((total, n) => total + n, 0);
  }
  sumVariadic(1, 2, 3) // evaluates to 6
  ```

  - 이렇게 사용하면 type을 추론하는 것도 가능해진다.





#### \# Rest parmeter 주의점 및 특징

- 함수는 최대 한개의 rest parameter를 가질 수 있다.
- 그리고 해당 rest parametersms 함수 파라미터 리스트에서 가장 뒤에 위치해야 한다.



- 예시

  - TypeScript의 `console.log`의 built-in declaration를 살펴보자.
  - `console.log`는 optional 메시지를 받아들이고 추가적인 argument를 개수 제한 없이 받아들이고 있다.

  ```ts
  interface Console {  
    log(message?: any, ...optionalParams: any[]): void
  }
  ```

  

### call, apply, and bind

- 함수를 괄호 `()`와 함께 호출하는 방법 외에도 JS는 다른 방법들을 지원한다.

  ```ts
  function add(a: number, b: number): number {
    return a + b;
  }
  
  add(10, 20);
  add.apply(null, [10, 20]);
  add.call(null, 10, 20);
  add.bind(null, 10, 20)();
  ```

- `apply`

  - 첫번째 인자를 함수 내의`this`에 바인딩시키고,
  - 두번째 인자를 함수의 파라미터에 spread시킨다.

- `call`

  - `apply`랑 똑같은 일을 한다.
  - 대신 `apply`는 배열을 넘겨준뒤 spread 시키지만, `call`은 파라미터를 그냥 순서대로 넘겨준다.

- `bind`

  - `bind`는 `apply`, `call`과 비슷하지만 얘는 함수를 호출하는게 아니라 `()`로 호출할 수 있게 함수 그 자체를 반환해준다.
  - `bind`는 (), .call 또는 .apply를 사용하여 호출할 수 있는 새 함수를 반환하고 원하는 경우 지금까지 바인딩되지 않은 매개 변수에 바인딩하기 위해 더 많은 인수를 전달합니다.



- **TSC Flag: strictBindCallApply**
  - `.call`, `.apply`, `.bind`를 안전하게 사용하려면 `tsconfig.json`에서 `strictBindCallApply` 옵션을 켜놓자.
  - `strict` 모드를 켜놓으면 자동으로 켜진다.



### Typing `this`

- JS에서는 `this`가 context에 따라 가리키는 객체가 달라진다.

  - 프로그램을 불안정하게 만드는 요소다.

- 이러한 이유로, 많은 개발 팀들은 클래스 내의 메서드에서 쓰는 `this` 키워드를 제외하고는 모두 금지시켜 버리고 있다.

  - 이는 `no-invalid-this` TSLint rule을 통해 적용할 수 있다.

  

- `this` 키워드가 불안정한 이유는 할당 방식과 관련이 있다.

- 일반적인 규칙은 메서드를 호출할 때 `this`가 `.` 옆에 있는 것을 가리킨다는 것이다.

  ```ts
  let x = {
  	a() {
      return this;
    }
  }
  
  x.a();  // 객체 x
  ```

- 그러나 `a()` 메서드를 호출하기 전에 변수에 재할당하면, 결과는 바뀌게 된다.

  ```ts
  let a = x.a;
  a();  // undefined
  ```

  

- 날짜를 포매팅하는 유틸리티 함수가 아래와 같이 생겼다고 해보자.

  ```ts
  function fancyDate() {
  	return `${this.getDate()}/${this.getMonth}/${this.getFullYear()}`
  }
  ```

  - 당신이 이 API를 초보 시절에 작성했다고 해보자.

  - `fancyDate()`를 사용하려면, 무조건 `Date` 객체를 `this`에 바인딩시켜야만 한다.

    ```ts
    fancyDate.call(new Date());  // "4/14/2005"
    ```

  - 근데 깜빡하고 `Date` 객체를 `this`에 바인딩시키는 것을 까먹으면, 런타임 exception이 터진다.

    ```ts
    fancyDate();  // Uncaught TypeError: this.getDate is not a function
    ```



- `this` 키워드가 당신이 함수를 호출하는 방식에 의존하여 작동을 달리한다는 사실은 프로그래밍을 어렵게 할 수 있다.

- 감사하게도, TS가 당신을 도와줄 수 있다.

  - 만약, 함수가 `this` 키워드를 사용할 때, `this`가 가질 것으로 기대되는 타입을 함수의 첫번째 파라미터에 선언하라. 

    - 무조건 맨 앞이어야 함.

  - 이렇게 하면 TS는 모든 호출에서 `this`가 가리키는 것을 지정해준다.

  - `this`는 다른 파라미터들과 다르게 취급된다

    - 함수 시그니처의 일부로 사용될 때 예약어로 취급된다.

    ```ts
    function fancyDate(this: Date) {
    	return `${this.getDate()}/${this.getMonth()}/${this.getFullYear()}`
    }
    ```

- 이렇게, `this` 키워드에 대한 타입을 지정해주고 나서 함수를 호출해보자.

  ```ts
  fancyDate.call(new Date) // "6/13/2008"
  fancyDate() // Error TS2684: The 'this' context of type 'void' is not assignable to method's 'this' of type 'Date'
  ```

  - 사실 호출할라고 하면 fancyDate()에 이미 빨간줄이 그어져있다.



- **TSC Flag: noImplicitThis**
  - `this` 키워드의 타입이 언제나 명시되어 있게 하기 위해서 `tsconfig.json`의 `noImplicitThis` 옵션을 켜놓자.
  - 만약 `strict` 모드가 켜져있으면 자동으로 켜진다.
  - `noImplicitThis`는 클래스나 객체의 함수에 대해 `this` annotation을 적용하지 않습니다.



### Generator Functions

- Generator 함수들(줄여서 generator)은 값 뭉텅이를 만들어내는 쉬운 방법을 제공한다.

- generator들은 이 generator들의 소비자가 값들이 생성되는 속도를 섬세하게 제어할 수 있도록 해준다.

  - 왜냐하면 얘네들은 lazy하기 때문이다.
  - 즉, 다음 값은 소비자가 요청을 할 때만 계산된다.

- generator가 쓰이기 좋은 사례

  - 무한 리스트

  ```ts
  function* createFibonacciGenerator() {
    let a = 0;
    let b = 1;
    while(true) {
      yield a;
      [a, b] = [b, a + b];
    }
  }
  
  let fibonacciGenerator = createFibonacciGenerator() // IterableIterator<number>
  fibonacciGenerator.next()   // evaluates to {value: 0, done: false}
  fibonacciGenerator.next()   // evaluates to {value: 1, done: false}
  fibonacciGenerator.next()   // evaluates to {value: 1, done: false}
  fibonacciGenerator.next()   // evaluates to {value: 2, done: false}
  fibonacciGenerator.next()   // evaluates to {value: 3, done: false}
  fibonacciGenerator.next()   // evaluates to {value: 5, done: false}
  ```

  - `function*`는 generator 함수를 나타낸다.
  - generator를 호출하면 iterable iterator가 반환된다.
  - 우리의 generator는 무한정 값을 뽑아낼 수 있다.
  - generator들은 `yield` 키워드를 사용해 값을 산출해낸다.
    - 소비자가 generator에게 다음값을 요구(`next` 호출)하면, `yield`는 소비자에게 결과값을 보내주고, 소비자가 다음 값을 요구하기 전까지 수행을 멈춘다.

- generator가 반환하는 타입을 명시적으로 나타낼 수도 있다.

  - generator가 `yields`하는 값을 `IterableIterator`로 래핑하면 된다.

  ```ts
  function* createNumbers(): IterableIterator<number> {
  	let n = 0;
    while(1) {
      yield n++;
    }
  }
  
  let numbers = createNumbers();
  numbers.next();  // {value: 0, done: false}
  numbers.next();  // {value: 1, done: false}
  numbers.next();  // {value: 2, done: false}
  ```

  

### Iterators

- generator가 값들의 stream을 생산해낼 때, <u>iterator를 통해 이 값들을 소비할 수 있다.</u>
- 용어 정리
  - `Iterable`
    - `Symbol.iterator`이라 불리는 프로퍼티를 포함하고 있는 모든 객체.
    - 이때, `Symbol.iterator`는 iterator를 반환하는 함수여야 한다.
  - `Iterator`
    - `next`라 불리는 메서드를 정의하는 모든 객체
    - 이때, `next`는 `value`와 `done`이라는 프로퍼티를 가진 객체를 반환해야 한다.

- generator를 만들었을 때(`createFibonacciGenerator`를 호출한다고 해보자), 당신은 iterable iterator를 반환받는다.

  -  왜냐하면 해당 객체는 `Symbol.iterator`와 `next` 메서드를 둘 다 정의하고 있기 때문이다.

- 당신은 `iterator`나 `iterable`을 `Symbol.iterator`나 `next` 메서드를 구현하는 객체(또는 클래스)를 만듦으로써 정의할 수 있다.

  - 예시 (1부터 10까지 반환하는 iterator)

    ```ts
    let numbers = {
    	*[Symbol.iterator]() {
        for(let n = 1; n <= 10; n++) {
          yield n;
        }
      }
    }
    ```

  - 위와 같이 작성하면, TS가 아래와 같이 타입을 추론해준다.

    ![image-20211107173730286](/Users/mojo/Library/Application Support/typora-user-images/image-20211107173730286.png)

  - 다시 말해, `numbers`는 `iterable`이고, generator 함수 `numbers[Symbol.iterator]()`를 호출하면 `iterable iterator`를 반환한다.

- 당신이 직접 own iterator를 정의할 수도 있고, JS의 공통 컬렉션 타입(Array, Map, Set, String 등등)을 위한 내장 `iterator`들을 사용할 수도 있다.

  ```ts
  // Iterate over an iterator with for-of
  for (let a of numbers) {
    // 1, 2, 3, etc.
  }
  
  // Spread an iterator
  let allNumbers = [...numbers];
  
  // Destructure an iterator
  let [one, two, ...rest] = numbers
  ```



### Call Signatures

- `sum` 함수

  ```ts
  function sum(a: number, b: number): number {
  	return a + b;
  }
  ```

- `sum` 함수의 type은 무엇인가?

  - 당연히 `sum`은 함수이므로, type 역시 `Function`이다.

- 그런데 이 `Function` 타입이라는 정보를 가지고 할 수 있는 것은 별로 없다.

- 모든 객체가 `object` 타입이라는 정보를 가지고 할 수 있는 것이 별로 없듯이 이 `Function` 타입이라는 정보는 당신에게 구체적인 정보를 알려주지 않는다.

- 위의 `sum` 함수를 type화 하는 다른 방법이 있을까?

  - `sum`은 두 개의 `number`를 받아서 `number`를 반환하는 함수다.

  - TS에서는 이러한 형태의 함수를 아래와 같이 type화 할 수 있다.

    ```ts
    (a: number, b: number) => number
    ```

- 이것이 TS에서 제공하는 함수의 타입을 위한 문법이다.

  - 이러한 문법을 **call signature**(또는 type signature)라고 부른다.

- 이러한 call signature는 화살표 함수와 비슷하다 - 이것은 의도된 것이다!

- 이 문법은 당신이 함수들을 인자로 넘겨주거나 다른 함수로부터 함수를 반환받을 때 이러한 함수들을 type화 하기 위해 사용하는 문법이다.

- 참고: `a`와 `b`라는 파라미터 이름은 아무 상관없다. 타입이 중요하다.



- 함수의 call signature는 오직 **type-level** code만을 포함한다.
  - 다시 말해, 값에 대한 정보는 포함하지 않고 오직 타입에 대한 정보만을 포함한다.
- 즉, 함수 call signature는 아래와 같은 것들을 표현할 수 있다.
  - 파라미터 타입
  - `this` 타입
  - 반환 타입
  - rest 타입
  - optional 타입
- 그리고 함수 call signature는 default values를 표현할 수 없다.
  - 왜냐면 default value는 value지, 타입이 아니니깐..
- 그리고 함수 call signature들은 body가 없기 때문에(TS는 body로부터 타입을 추론할 수 있다.) call signature들은 반환타입을 명시적으로 표시해야 한다.



> **Type Level and Value Level Code**
>
> - 사람들은 정적 타입을 사용하는 프로그래밍에 관해 얘기할 때, "type-level"과 "value-level"이라는 용어를 많이 사용하곤 한다.
>   - 이 용어들에 대해 알아보자.
> - 이 책에서 **type-level** 코드라는 단어를 사용할 때, 저자가 말하는 type-level 코드란, 오직 타입과 타입 연산으로만 구성된 코드를 말한다.
> - **value-level** 코드는 반대로 type-level 코드에 해당하지 않는 모든 것을 말한다.
> - A rule of thumb(경험 법칙, 직접 해보고 깨닫는 것들)에 의하면, 
>   - 만약 어떤 코드가 유효한 JS 코드라면, 그것은 value-level 코드다.
>   - 만약 유효한 TS 코드이지만 유효한 JS 코드가 아니라면, 그것은 type-level 코드다.



### Contextual Typing

- 예시

  ```ts
  type Log = (message: string, userId?: string) => void
  
  let log: Log = (
  	message,
    userId = 'Not signed in'
  ) => {
    let time = new Date().toISOString();
    console.log(time, message, userId);
  }
  ```

  - 위 예에서 함수의 파라미터와 반환값에 type에 대해 명시하지 않아도 TS는 함수의 call signature를 통해 type을 추론할 수 있다.
  - 이런 TS의 강력한 타입 추론 기능을 **contextual-typing**이라고 부른다.

- 예시 2

  ```ts
  function times(
  	f: (index: number) => void,
    n: number
  ) {
      for(let i = 0; i < n; i++) {
        f(i);
      }
  }
  ```

  - `times(n => console.log(n), 4)`

  - TS는 context로부터 `n`이 `number`임을 추론할 수 있다.

    - 우리는 `f`의 인자 `index`가 `number`임을 `times` 함수의 call signature에서 명시했으므로, 

      똑똑한 TS가 `n`이 `index`에 해당함을 추론하고 `number` 타입이어야 함을 알아낸다.

  - 만약 우리가 `f` 함수를 inline으로 선언하지 않았다면, TS는 타입을 추론할 수 없다.

    ```ts
    function f(n) {  // Error TS7006: Parameter 'n' implicity has an 'any' type
    	console.log(n);
    }
    times(f, 4);
    ```

  

### Overloaded Function Types

- 이전 섹션에서 사용한 함수 타입 문법  `type Fn = (...) => ...`는 `shorthand call signature`다.

- 우리는 이 시그니처를 더욱 명시적으로 표현할 수 있다.

- 다시 한번 `Log` 함수의 예를 들어보자.

  ```ts
  // Shorthand call signature
  type Log = (message: string, userId?: string) => void
  
  // Full call signature
  type Log = {
    (message: string, userId?: string): void
  }
  ```

  - 두 방법은 문법 형태만 다르고 완전히 동일한 방법이다.
  - 따라서 간단한 경우에는 shorthand 방법을 쓰는게 좋다.
  - 그런데, 훨씬 복잡한 함수들에 대해서는 full signature를 쓰는게 좋은 경우들이 있다.

- 이렇게 full signature를 쓰는게 더 좋은 경우 중 하나가 **함수 타입을 오버로딩하는 경우**다.

  - 하지만 그전에, 함수를 overload 한다는 것이 무슨 뜻인가?
  - Overloaded function이란 여러 개의 call signature를 가진 함수를 말한다.

- 대부분의 PL에서는, 한번 당신이 파라미터와 반환 타입을 정해서 함수를 선언한 후

  정확히 정의한 대로 파라미터를 넘겨주면, 언제나 정의한 타입을 가지는 반환값을 돌려받게 된다.

- 그러나 JS에서는 그렇지 않다.

- JS는 dynamic한 언어이기 때문이다.

  - JS에서 주어진 함수를 호출하는 방법이 여러가지 존재하는 것은 흔한 일이다.
  - 그뿐만 아니라 때로는 반환 타입이 입력되는 인자의 타입에 따라 달라지기도 한다.

- TS는 이러한 dynamism(overloaded 함수 선언, 입력 값의 타입에 의존하는 출력값 타입)을 static type system을 통해 모델링한다.

  - 우리는 이러한 기능을 당연히 여길 수 있지만, 이런 기능은 type system이 가질 수 있는 정말 고급 기능이다.

- 당신은 overloaded 함수 시그니처를 통해 매우 expressive한 API를 설계할 수 있다.

- 예를 들어, 휴가를 예약하는 API를 설계한다고 해보자. - 그리고 그것을 `Reserve`라 부르자.

  - 해당 API의 타입을 생각해보면서 시작하자

  ```ts
  type Reserve = {
  	(from: Date, to: Date, destination: string): Reservation
  }
  ```

  ```ts
  let reserve: Reserve = (from, to, destination) => {
  	// ...
  }
  ```

- 우리는 여기서 편도 여행을 지원하는 API를 추가할 수도 있을 것이다.

  ```ts
  type Reserve = {
  	(from: Date, to: Date, destination: string): Reservation
  	(from: Date, destination: string)
  }
  ```

  - 만약 당신이 이 코드를 실행하려 하면, TS가 `Reserve`를 구현하는 부분에서 에러를 던지는 것을 발견할 것이다.

  ![image-20211107201908764](/Users/mojo/Library/Application Support/typora-user-images/image-20211107201908764.png)

- 이는 TS에서 call signature overloading이 동작하는 방식과 연관이 있다.

- 만약 당신이 함수 `f`에 대해 여러 개의 overloaded signature들을 선언했다고 해보자.

  - 함수 호출자의 관점에서 `f`의 타입은 이러한 overload signature들의 union(합집합)이다.
  - 그러나 `f` 함수의 구현체의 관점에서 보면, 실제로 구현될 수 있는 하나의 결합된 타입이 필요하다.
  - 당신이 `f` 함수를 구현할 때는 이 combined call signature를 manually 선언해야 한다.
    - 이것은 TS가 유추해주지 않는다.

- 우리의 `Reserve` 예시에서, `reserve` 함수를 아래와 같이 수정해보자.

  ```ts
  type Reserve = {
  	(from: Date, to: Date, destination: string): Reservation
    (from: Date, destination: string): Reservation
  } // 두 개의 overloaded 함수 시그니처를 선언
  
  let reserve: Reserve = (
  	from: Date,
    toOrDestination: Date | string,
    destination?: string
  ) => {  
    // ... 
  }
  // 구현체의 시그니처는 우리가 직접 Signature1 | Signature2를 계산한 것이다.
  // 결합된 시그니처는 reserve를 호출하는 함수에게는 보이지 않음을 주의하라.
  ```

  - 함수를 호출하는 자의 입장에서의 `Reserve`의 시그니처는 아래와 같다.

    ```ts
    type Reserve = {
    	(from: Date, to: Date, destination: string): Reservation
      (from: Date, destination: string): Reservation
    } 
    ```

  - 아래와 같은 시그니처가 우리가 만든 결합된 시그니처를 포함하지 않음에 주의하라.

    ```ts
    // Wrong!
    type Reserve = {
    	(from: Date, to: Date, destination: string): Reservation
      (from: Date, destination: string): Reservation
      (from: Date, toOrDestination: Date | string, destination?: string): Reservation
    }
    ```

- `reserve`가 두 가지 방법 중 하나로 호출되기 때문에, 당신이 `reserve`를 구현할 때는 두 가지 중 어떻게 호출되었는지 체크해야 한다.

  ```ts
  let reserve: Reserve = (
  	from: Date,
    toOrDestination: Date | string,
    destination?: string
  ) => {  
    if (toOrDestination instanceof Date && destination !== undefined) {
      // Book a one-way trip
    } else if (typeof toOrDestination === 'string') {
      // Book a round trip
    }
  }
  ```

  

- **Keeping Overload Signatures Specific**

  - 일반적으로, overloaded function type을 선언할 때, 각각의 overload 시그니처(`Reserve`)는 구현체의 시그니처(`reserve`)에 할당 가능해야 한다.

  - 때문에, 구현체의 시그니처를 선언할 때 너무 general하게 선언해버리는 경우가 생길 수 있다.

    ```ts
    let reserve: Reserve = (
    	from: any,
      toOrDestination: any,
      destination?: any
    ) => {
      // ...
    }
    ```

    - 예를 들면, 위와 같이 작성해도 코드는 돌아간다.
    - 하지만, overload 기능을 사용할 때는 구현체의 signature를 가능한 한 구체적으로 유지해라.
    - 이렇게 하는게 함수를 구현하기 더 쉽게 해준다.







## Polymorphism



### When Are Generics Bound?



### Where Can You Declare Generics?



### Generic Type Inference



### Generic Type Aliases



### Bounded Polymorphism



### Generic Type Defaults



## Type-Driven Development





