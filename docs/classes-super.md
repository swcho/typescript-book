#### `super`

자식(`Child`) 클래스에서 `super`를 통해서 부모(`Base`) 클래스의 함수를 호출할 수 있습니다.

```ts
class Base {
    log() { console.log('hello world'); }
}

class Child extends Base {
    log() { super.log() };
}
```

이는 다음과 같이 부모(`Base`) 클래스 객체의 `prototype`을 통해 호출하는 코드를 생성합니다.

```js
var Base = (function () {
    function Base() {
    }
    Base.prototype.log = function () { console.log('hello world'); };
    return Base;
})();
var Child = (function (_super) {
    __extends(Child, _super);
    function Child() {
        _super.apply(this, arguments);
    }
    Child.prototype.log = function () { _super.prototype.log.call(this); };
    return Child;
})(Base);

```

클래스 맴버 변수에는 `super` 대신 `this`를 사용해야 합니다.

물론 자식(`Child`) 클래스에서 `this` 키워드를 통해서 부모(`Base`) 클래스의 `log()` 함수를 사용할 수 있습니다.
이 경우, `Child` 클래스에서는 호출하고자 하는 부모(`Base`) 클래스의 `log()` 함수와는 다른 이름을 사용(예제 에서는 `logWorld()`)해야 합니다. 그러면 JavaScript의 prototype chain을 통해 자연스럽게 `Base` 클래스의 `log()` 함수를 호출하게 되겠죠.

```ts
class Base {
    log = () => { console.log('hello world'); }
}

class Child extends Base {
    logWorld() { this.log() };
}
```

TypeScript 컴파일러는 `super`를 잘못 하용하였을 경우, 다음과 같이 경고를 해 줍니다.

```ts
module quz {
    class Base {
        log = () => { console.log('hello world'); }
    }

    class Child extends Base {
        // ERROR : only `public` and `protected` methods of base class are accessible via `super`
        logWorld() { super.log() };
    }
}
```
