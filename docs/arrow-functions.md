* [Arrow Functions](#arrow-functions)
* [Tip: Arrow Function Need](#tip-arrow-function-need)
* [Tip: Arrow Function Danger](#tip-arrow-function-danger)
* [Tip: Libraries that use `this`](#tip-arrow-functions-with-libraries-that-use-this)
* [Tip: Arrow Function inheritance](#tip-arrow-functions-and-inheritance)

### Arrow Functions

사랑 스럽게도 *fat arrow* `=>` 라고도 불립니다. 다른 언어에서는 *lambda function* 이라고도 알려 져 있죠.
*fat arrow*는 다음과 같은 이유료 탄생했습니다.
1. `function`을 다 입력하지 않아도 됩니다.
1. `this`의 의미를 문법적으로 적절히 해석해 줍니다.
1. `arguments`의 의미를 문법적으로 적절히 해석해 줍니다.

함수형 언어라 하는 것들은 함수를 자주 선언하게 마련입니다. JavaScript에서는 `function` 키워드를 통해 함수를 선언하게 되죠. `fat arrow`는 이러한 선언을 조금이라도 간략하게 해 줍니다.
```ts
var inc = (x) => x + 1;
```

처음 JavaScript를 배울 때, `this` 키워드의 개념을 잡는 것이 쉽지 않습니다. 다른 언어에서 클래스를 정의하고 그 내부에서 사용하는 `this`는 통상 해당 클래스의 자기 자신의 객체를 가리킵니다. 하지만, JavaScript 에서는 호출한 객체가 될 수도 있으며, `call`, `apply` built in 함수등을 통해 `this`가 가리키는 객체를 변경할 수도 있습니다.

다음의 순수한 JavaScript로 정의한 클래스 객체를 예로 들어 보겠습니다.

```ts
function Person(age) {
    this.age = age;
    this.increaseAge = function() {
        this.age++; // 여기에서의 this는 항상 Person 객체의 인스턴스 일까요?
    }
}

var person = new Person(1);
// 여기서 person.age는 1 입니다.

setTimeout(person.increaseAge,1000);
// 1초 뒤에 수행한 increaseAge 로 인해 person.age가 2가 될까요?

setTimeout(function() { console.log(person.age); },2000);
// 2초 뒤에 수행한 결과는 1 입니다.
```

만일, 위의 코드를 브라우저에서 수행하면, `increaseAge` 함수의 `this`는 `window` 객체를 가리키게 됩니다. 이는 `increaseAge`를 호출한 객체가 `window`이기 때문입니다.

이제 arrow function을 사용한 예제 입니다.

```ts
function Person(age) {
    this.age = age;
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000);
// 이제는 2 값으로 출력 합니다.
```
위와 같이 동작하는 이유는 arrow function 에서 사용한 `this`는 실제로 arrow function 밖에서 참조한 `this`이기 때문 입니다.
다음은 위의 코드를 [TypeScript로 컴파일한 결과](https://www.typescriptlang.org/play/index.html#src=function%20Person(age)%20%7B%0D%0A%20%20%20%20this.age%20%3D%20age%3B%0D%0A%20%20%20%20this.growOld%20%3D%20()%20%3D%3E%20%7B%0D%0A%20%20%20%20%20%20%20%20this.age%2B%2B%3B%0D%0A%20%20%20%20%7D%0D%0A%7D%0D%0Avar%20person%20%3D%20new%20Person(1)%3B%0D%0AsetTimeout(person.growOld%2C1000)%3B%0D%0A%0D%0AsetTimeout(function()%20%7B%20console.log(person.age)%3B%20%7D%2C2000)%3B) 입니다.
```ts
function Person(age) {
    var _this = this;
    this.age = age;
    this.growOld = function () {
        _this.age++;
    };
}
var person = new Person(1);
setTimeout(person.growOld, 1000);

setTimeout(function () { console.log(person.age); }, 2000);
```

TypeScript로 작성하면 `class` 키워드를 사용해서 조금더 멋진 코드를 작성할 수 있습니다.

```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

> [관련 동영상 🌹](https://egghead.io/lessons/typescript-make-usages-of-this-safe-in-class-methods)

#### Tip: Arrow Function Need

다음과 같이 다른 호출 컨텍스트에서 함수를 호출하고자 한다면 arrow function 대신  `function` 을 사용해야 합니다.

```ts
var growOld = person.growOld;
// Then later someone else calls it:
growOld();
```

호출 컨텍스를 선언한 객체 자신에서만 유지하고자 할때만, arrow function을 사용합니다.
```ts
person.growOld();
```
`growOld` 함수를 arrow function으로 정의했다면, `growOld` 함수 내에서 참조하는 `this`는 `person`이 됩니다.

#### Tip: Arrow Function Danger

만일 정의하는 함수에서의 `this`의 calling context를 변경하고자 한다면 arrow function 보다는 `function`으로 함수를 정의해야 합니다. 또한 arrow function 에서는  `arguments`는 사용할 수 없다.

#### Tip: Arrow functions with libraries that use `this`
`jQuery`를 포함한 여러 라이브러리들 중 `jQuery.each`와 같은 반복 가능한 함수(interables) 들은 `this`를 현재의 값으로 설정해서 함수를 호출 합니다.
따라서, 이경우 arrow function을 주의해서 사용해야 합니다.

예를 들어 아래와 같이 코딩을 하면,
```ts
let _self = this;
$('li').each(() => {
    console.log(_self);
    console.log(this);
});
```

아래와 같은 코드를 생성합니다.
```ts
var _this = this;
var _self = this;
something.each(function () {
    console.log(_self);
    console.log(_this);
});
```

즉 `this`가 함수를 정의한 외부 컨텍스트를 가지게 되는 것이죠.

jQuery를 이용해 개발자들이 원하는 결과를 얻으려면 다음과 같이 작성해야 합니다.
```ts
let _self = this;
$('li').each(function() {
    console.log(_self); // the lexically scoped value
    console.log(this); // the library passed value
});
```

#### Tip: Arrow functions and inheritance

arrow function을 통해 정의한 member 함수는 사실 상 member 변수 입니다.
다시말하면, 함수형의 변수에 arrow function을 사용하여 anonymous function을 선언 과 동시에 대입을 하는 것 입니다.

다음의 예를 살펴 봅니다.
```ts
class Adder {
    constructor(public a: number) {}
    add = (b: number): number => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    add = (b: number): number => {
        return super.add(b); // 에러 발생
    }
}
```

위의 코드는 사실상, 자식 클래스(`ExtendedAdder`) 가 부모 클래스(`Adder`)의 함수(`add`)를 overriding 을 한다기 보다는, 부모 클래스의 `add` 맴버 변수에 새로운 anonymous 함수를 정의해서 대입하는 것이죠.

따라서 위와같이 `super.add(b)`와 같은 성립하지 않음으로 컴파일 에러가 발생합니다.

`super` 를 사용하고자 한다면 다음과 같이 member 함수로 선언하면 됩니다.

```ts
class Adder {
    constructor(public a: number) {}
    add(b: number): number {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    add(b: number): number {
        return super.add(b);
    }
}
```