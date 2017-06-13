### Classes

The reason why it's important to have classes in JavaScript as a first class item is that:

JavaScript에 class 가 필요한 이유는 다음과 같습니다.  
1. [클래스는 유용한 구조적인 추상화를 제공합니다.](./tips/classesAreUseful.md)  
1. 여러 프레임워크나 방법론에서 제안하는 각기 다른 방식의 클래스 사용 방식에서 일관된 사용방법을 제공합니다.
1. 객체 지향 개발자들은 이미 클래스를 이해하고 있습니다.

드디어 JavaScript 개발자들은 `class`를 사용할 수 있습니다.
TypeScript로 정의한 Point 클래스 예제를 살펴 보겠습니다.

```ts
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // {x:10,y:30}
```

위의 코드는 다음과 같은 ES5 코드를 생성합니다.

```ts
var Point = (function () {
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    Point.prototype.add = function (point) {
        return new Point(this.x + point.x, this.y + point.y);
    };
    return Point;
})();
```

TypeScript는 이렇게 고전적인 방식으로 class를 사용했던 패턴을 생성해 줍니다.

### Inheritance (상속)

TypeScript 에서는 `extends` 키워드를 통해 _단일_ 상속을 지원합니다.
아래의 예제는 `Point` 클래스를 상속 받아 `Point3D`로 확장을 하는 코드 예제 입니다.

```ts
class Point3D extends Point {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    add(point: Point3D) {
        var point2D = super.add(point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    }
}
```

만일 부모 클래스 즉 `Point`가 생성자(`constructor`)를 가진다면, 자식 클래스인 `Point3D`에서는 `super`를 통해 반드시 부모 클래스의 생성자를 호출하도록 TypeScript가 컴파일 에러를 통해 강제 합니다.
`super`를 통해 부모 클래스의 생성자를 호출하고 나면, 나머지 초기화 과정에 대한 코드를 작성할 수 있습니다.
`Point3D`의 경우 `z` 값을 초기화 합니다.

여기서 주목할 것은, 부모 클래스인 `Point`의 함수은 `add`를 override 했다는 것 입니다. 그리고 부모 클래스의 함수를 `super.` 키워드를 통해 접근할 수 있습니다.

### Statics (정적 변수)

모든 클래스의 인스턴스 사이에서 공유할 수 있는 `static` 변수를 지원합니다.

```ts
class Something {
    static instances = 0;
    constructor() {
        Something.instances++;
    }
}

var s1 = new Something();
var s2 = new Something();
console.log(Something.instances); // 2
```

동일한 방식으로 정적 함수도 정의할 수 있습니다.

### Access Modifiers (접근제어자)

TypeScript 는 객체지향 언어에서 사용하는 동일한 접근 제어자를 지원합니다.

| accessible on | `public` | `protected` | `private` |
| --- | --- | --- | --- |
| class | yes | yes | yes |
| class children | yes | yes | no |
| class instances | yes | no | no |

접근 제어자를 생략하면 원래의 JavaScript가 그렇듯 `public` 이 됩니다.

이는 TypeScript 컴파일러가 접근 제어가 가능한 코드로 JavaScript를 생성한다는 것을 의미하지 않습니다.
다만, 컴파일 과정을 통해서 접근 제어에 위반한 코드에 대해 컴파일 에러를 통해 접근 제어를 하는 것을 의미 합니다.

[역주]
실제로 `private`의 경우 IFFE 안에 변수를 정의하는 식으로 구현할 수 있겠지만, `protected`의 경우, JavaScript 언어 스펙 상 불가능한 내용이겠죠.

```ts
class FooBase {
    public x: number;
    private y: number;
    protected z: number;
}

// EFFECT ON INSTANCES
var foo = new FooBase();
foo.x; // okay
foo.y; // ERROR : private
foo.z; // ERROR : protected

// EFFECT ON CHILD CLASSES
class FooChild extends FooBase {
    constructor() {
      super();
        this.x; // okay
        this.y; // ERROR: private
        this.z; // okay
    }
}
```

위의 예제는 맴버 변수만 사용했지만, 맴버 함수도 가능합니다.

### Abstract (추상 키워드)

`abstract` 키워드와 함께 정의한 __추상 클래스__는 다음과 같은 특징을 가집니다.

* `abstract` **classes** 는 직접 생성할 수 없으며, 상속 받은 자식 클래스를 통해 생성 후 사용가능하다.
* `abstract` **members** 는 자식 클래스에서 반드시 구현해야 한다.

### Constructor is optional (생성자는 없어도 됩니다.)

The class does not need to have a constructor. e.g. the following is perfectly fine.

```ts
class Foo {}
var foo = new Foo();
```

### Define using constructor

Having a member in a class and initializing it like below:

```ts
class Foo {
    x: number;
    constructor(x:number) {
        this.x = x;
    }
}
```

위의 코드든 다음과 같이 생성자에 접근 제어자를 명시하는 방식으로 간략하게 구현할 수 있습니다.

```ts
class Foo {
    constructor(public x:number) {
    }
}
```

### Property initializer

앰버 변수에 초기값은 다음과 같이 지정할 수 있습니다.

```ts
class Foo {
    members = [];  // Initialize directly
    add(x) {
        this.members.push(x);
    }
}
```
