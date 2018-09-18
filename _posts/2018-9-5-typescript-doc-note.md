---
layout: post
title: Typescript 文档读书笔记
date: 2018-09-05
categories: 前端
tags: typescript
---
{::options syntax_highlighter_opts="default_lang: typescript" /}

Typescript 文档读过几次, 每次读了就忘, 尝试做下笔记, 把书读薄, 看看有没有用。

# 基本类型

## 原始类型
`boolean`, `number`, `string`, `object`, `null`, `undefined`, `symbol`

## 数组
```
let list: number[] = [1, 2, 3];        // c 风格
let list: Array<number> = [1, 2, 3];   // 泛型风格
```

## 元组 Tuple

```
let x: [string, number]
x = ['hello', 10] // initialize
console.log(x[0], x[1]) // access

x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型
x[6] = true; // Error, 布尔不是(string | number)类型
```
> 这种越界访问的意义到底何在？ 把元组似乎变成了一个联合类型的数组？

## 枚举

```
// 索引默认从0开始, 加 1 自增
enum Color {
  Red,        // 0
  Green,      // 1
  Blue        // 2
}

// 指定值
enum Color {
  Red = 1,
  Green = 2,
  Blue = 4
}

// 数字枚举支持从数字反查
console.log(Color[4]) // output Blue

// 字符串枚举, 不支持反查
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}

// 动态枚举值
enum E {
  A = getSomeValue(), // A 动态生成
  B,                  // error！ B 一定要显示的赋值
}
```

### 常量字面量的枚举类型的成员也是类型

```
enum ShapeKind {
    Circle,
    Square,
}

let kind: ShapeKind.Circle = ShapeKind.Circle
kind = ShapeKind.Square // 类型不兼容
kind = 100 // 但是赋值任意的数字给 kind，均不报错. 😂 为啥呢？
```
也就是说 `ShapeKind.Circle` 即是值，又是类型。

### 运行时的枚举
ts 转为 js 后，（非 const）枚举显然是以对象的形式存在的。所以 ts 让枚举可以兼容对象:

```
enum E {
  X, Y, Z
}

function f(obj: { X: number }) {
  return obj.X;
}

f(E)
```

### const 枚举
这里的 `const` 含义并不代表枚举的值不能修改（实际上无论有没有这个 const，枚举的值都不能修改）。这里的 `const` 是给 ts 编译器一个信息: 这个枚举里都是常量，不要把这个枚举存在对象里了，把他们的直接内敛到代码里去:

```
const enum E {
  X, Y = X * 2, Z
}

function f(e: E) {
  return e;
}

f(E.X)
f(E.Y)
f(E.Z)
```

转移成 js 后为:

```
function f(e) {
    return e;
}
f(0 /* X */);
f(0 /* Y */);
f(1 /* Z */);
```
所以这个是一个性能优化的方法。

### 外部枚举
[文档](http://www.typescriptlang.org/docs/handbook/enums.html#ambient-enums)的字认识，但是不明白啥意思.

## any
在对现有代码进行改写的时候，any类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。 你可能认为Object有相似的作用，就像它在其它语言中那样。 但是Object类型的变量只是允许你给它赋任意值 - 但是却不能够在它上面调用任意的方法，即便它真的有这些方法：

```
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

## void
一般用于表示函数没有返回值:
```
function warnUser(): void {
    alert("This is my warning message");
}
```

## null 和 undefined
这 2 种类型分别只有1个取值，就是它们自己。且默认情况下 null 和 undefined 是所有类型的子类型, 所以它们可以被赋值给任意类型。

## never
表示永不存在的值，一般用于表示函数不会到达终点（会抛出异常，或者死循环）

```
  // 返回never的函数必须存在无法达到的终点
  function error(message: string): never {
      throw new Error(message);
  }

  // 返回never的函数必须存在无法达到的终点
  function infiniteLoop(): never {
      while (true) {
      }
  }
```

## 类型断言（强制类型转换）

2 种形式:
```
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;

let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```
`as` 看起来更现代一点, 😂.

## 函数类型
```
function Test(x: number, y: number): number { return x + y; }

let myAdd: (baseValue: number, increment: number) => number = Test  // 1
let myAdd1: { (x: number, y: number): number } = Test               // 2
```
写法 1 比较常用, 写法 2 对应于 通过 interface 定义函数，即 [接口](#接口) 一节里介绍的**函数类型**。

### 构造函数类型
> 构造函数类型，亦称类类型

```
class Test {
    constructor(x: string) {}
}

let ctor: typeof Test = Test                   // 1
let myAdd1: { new(x: string): Test } = Test    // 2
```
写法 1 比较常用, 也简洁。写法 2 常用于[构造构造函数泛型](#在泛型里使用构造函数类型)。


# 接口

“鸭式辨型法”, “结构性子类型化”, 一般情况下只要结构吻合, 则认为类型兼容

```
// 定义接口
interface SquareConfig {
  width: number;                                 // 必须的类型
  color?: string;                                // 可选类型
  readonly x: number;                            // 只读类型
  [index: number | string]: any;                 // 可索引类型
  (source: string, subString: string): boolean;  // 函数类型
  setTime(d: Date)                               // 函数方法
}
```

其中**函数类型**用于表示该类型是一个函数. **函数方法**用于定义类的成员函数。

**函数方法**是很容易理解的，和一般语言是类似的。但是**函数类型**的这种声明看起来很让人困惑, 它实际上等价于用 type 声明了一个函数类型的别名:

```
interface Type {
    (x: number, y: number): void // Type 是一个函数，且符合这个声明
}
```

等价于:
```
type Type = (x: number, y: number) => void
```

上面2中声明，都能让下面的这个赋值合法:
```
let z: Type = function (x: number, y: number) { }
```

问题是 type 定义明明清楚明白，干嘛要搞个**函数方法**呢？这可能也是无奈之举了，js 里的变量很有可能既是函数，又是包含属性的对象，而且这在 js 开发中是大量使用的。例子:

```
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { return '' };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

## 额外的属性检查
**对象字面量**会被特殊对待而且会经过额外属性检查

```
interface SquareConfig {
    color: string;
    width?: number;
}

function createSquare(config: SquareConfig): void {
}

createSquare({ color: "red", width: 100, property: 1 });
      // 报错, 类型不兼容, property 是一个接口里没有声明的额外属性

let variable = { color: "red", width: 100, property: 1 }
createSquare(variable) // 赋值给一个变量，类型兼容了
```

> 这个特性被搞出了，是专门针对 **对象字面量** 的么，来搞我们程序员的么？心里 mmp

## 类静态部分与实例部分的区别
定义一个类包含2部分: 即静态部分和实例部分, 构造函数和静态方法都属于类的静态部分，而当去实现一个接口的时候你是在约束你的实例部分，所以下面这样的写法是不对的:

```
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

上面这个官方给的例子里 interface 的这个声明，即不符合**函数类型**的定义格式，也不符合**函数方法**的定义格式，从其文档上下文的推测来看，这是一种特殊的用法，应该属于**函数类型**的定义格式，表明这个 interface 声明的是一个构造构造函数。

```
interface ICoordinate {
    x: string;
    y: string;
}

class Coordinate implements ICoordinate {
    constructor(public x: string, public y: string) {
    }
}
```

这是标准的实现接口的方法, 如果用 es5 来写这个构造函数，则为:

```
function Coordinate(x, y) {
    this.x = x;
    this.y = y;
}
```
Coordinate 是类名，但本质上还是是个函数.

```
// 定义一个描述构造函数的 interface
interface CoordinateConstructor {
    new (x: string, y: string): Coordinate
}

// 这个赋值合法
let ctor: CoordinateConstructor = Coordinate
```

## 继承于类的接口

接口可以通过继承类来获得，（😂 这特性没在其他语言里见过），而且这里还有个特别的地方，当继承的类有 private 或者 protected 属性时，ts 的 `鸭式辨型法` 进化成了和 C#/java 等语言的名义类型 (nominal typing) 了。

```
class Control {
  protected state: any;
}

interface SelectableControl extends Control {
  select(): void;
}

class Button extends Control implements SelectableControl {
  select() { }
}

class MyImage implements SelectableControl { // 错误
  protected state: any;
  select() { }
}
```
上例中，`MyImage` 虽然实现了 `SelectableControl` 接口的所有方法 (`select`) 并声明了所有的属性 (`state`)，但是 `state` 属性在 `Control` 类里是 protected 的, 这时 ts 的 typing 系统就会变得严格起来，它要求 `state` 一定要是 `Control` 里声明的那个, 也就是说 `MyImage` 一定要通过继承 `Control` 才能拥有这个属性。

> 这特么，也是有意来搞人的把。😂

# 类

```
class Greeter {
    static sProperty: Date = new Date();                   // 共有静态变量
    private static sProperty1: Date;                       // 私有静态变量
    greeting: string;                                      // 属性声明
    private readonly property1: number;                    // 只读属性，在构造函数里初始化
    readonly numberOfLegs: number = 8;                     // 直接初始化的只读属性
    constructor(message: string, public date: string) {    // 构造函数，date 为参数属性
        this.greeting = message;
        this.property1 = 9;
    }
    greet() {
        Greeter.sProperty = new Date('2018-9-5');
        return `Hello ${this.greeting} on ${this.date}`;
    }
    static sMethod() {
        console.log(Greeter.sProperty);
    }
}


Greeter.sMethod()
let greeter = new Greeter("world", '2018-9-6');
console.log(greeter.greet()); // hello world on 2018-9-6
Greeter.sMethod()
```

* 和其他语言一样的 private，protect, public, static 修饰符，public 是默认
* 参数属性: 这个特性是在构造函数的参数上加上**修饰符**, 声明和赋值二合一, 算个语法糖（我喜欢😍）
* 构造函数使用 constructor 关键字作为函数名, 成员变量通过 `this` 来访问

## 继承

* 继承类一定要在构造函数里通过 `super` 关键字去调用父类的构造函数
* 继承类可以通过 `super` 关键字去调用父类的方法

```
class Animal {
    constructor(public name: string) { }
    move(distanceInMeters: number = 0) {
      console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) {
      super(name);                    // 调用父类的构造函数
    }
    move(distanceInMeters = 5) {
      console.log("Slithering...");
      super.move(distanceInMeters);   // 通过 super 调用父类方法
    }
}
```

## 存取器
记一下语法:

```
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

## 抽象类
和其他语言类似，记一下语法。

```
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
}
```

## 类的构造函数类型
类本身的类型（类的构造函数类型）是啥呢: `typeof [ClassName]`

```
class Greeter {
    static standardGreeting = "Hello, there";
}

let Construct: typeof Greeter = Greeter;
```

# 函数

```
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```

## 可选参数和默认参数
默认参数可以认为是一种特别的可选参数

```
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}
```

他们共享同一个类型，即：

```
(firstName: string, lastName?: string) => string
```

## 剩余参数
和 es6 区别不大, 注意类型的格式

```
let buildName: (firstName: string, ...restOfName: string[]) => string =
    function (firstName: string, ...restOfName: string[]) {
        return firstName + " " + restOfName.join(" ");
    }
```

## 函数重载
这个所谓的**函数重载**和一般语言里的**函数重载**在意义上似乎完全不同。这里的**函数重载**是为了解决**同一个函数传入不同类型的参数返回不同类型的值**的问题。语法也很清奇:

```
function getNumber(n: number): number;  // 重载1
function getNumber(n: string): string;  // 重载2
function getNumber(n: any): any {       // 兼容重载1 和 重载2 的一个声明, 本身不算重载
    return n;
}

let a: number = getNumber(1)       // a 的类型是 number
let b: string = getNumber('1')     // b 的类型是 string
let c: boolean = getNumber(true)   // getNumber 不能接受除了 number 和 string 意外的参数
```
> 这种重载，也可以用在类里成员函数和静态函数都可以
> 这就叫重载，你气不气？

常规意义上的重载是不允许的, 需要用泛型去处理下面的这种情况:

```
function getNumber(n: number): number {}

function getNumber(n: string): string {} // error: duplicate function implement
```

# 泛型
和其他语言类似:

```
let identity: <T>(arg: T) => T =
    function <T>(arg: T): T {
        return arg;
    }
```
或者
```
let identity: {<T> (arg: T): T} =
    function <T>(arg: T): T {
        return arg;
    }
```
> `<T>(arg: T) => T` 和 `{<T> (arg: T): T}` 等价

用 interface 来定义这个函数类型:
```
interface GenericIdentityFn {
    <T>(arg: T): T
}

let identity: { <T>(arg: T): T } =
    function <T>(arg: T): T {
        return arg;
    }

let a: GenericIdentityFn = identity
identity<number>(1)     // 明确泛型为 number
identity<string>(1)     // 类型不匹配
identity('1')           // 类型推断
```

将泛型语法函数转为非泛型函数：

```
let numberIdentity: GenericIdentityFn<number> = identity;
let stringIdentity: GenericIdentityFn<string> = identity;
```

## 泛型类
与一般语言，并无太多区别, 如果你熟悉 java 或者 C# 的泛型, ts 的就是个简化版。

```
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
```

## 泛型约束

看例子秒懂，对吧。

```
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

## 在泛型约束中使用泛型参数

```
function getProperty<T>(obj: T, key: keyof T) {
    return obj[key];
}

let obj = {a: 1, b: 2};

getProperty(obj, 'a') // OK, 'a' 可以被赋予联合类型 'a' | 'b'
getProperty(obj, 'c') // error, 'c' 不能被赋予联合类型 'a' | 'b'
```

`keyof [泛型约束]` 代表着 T 类型对象所有 key 的联合类型, 比如上例中的这次调用，第二个参数的类型是 `'a' | 'b'`, 其完整的函数定义为:

```
(obj: { a: number, b: number }, key: "a" | "b") => number
```

## 在泛型里使用构造函数类型
> 构造函数类型，亦称类类型

使用的是 [构造函数类型](#构造函数类型) 声明的**写法2**。
```
function create<T>(c: {new(): T}): T {
    return new c();
}
```

# 类型推导
最佳通用类型原则: 看当前的语句，找出所有候选类型里最佳的那个类型.

```
class Animal { }
class Rhino extends Animal { }
class Elephant extends Animal { }
class Snake extends Animal { }

let zoo = [new Rhino(), new Elephant(), new Snake()];
```
上例中 `zoo` 里元素的类型有: `Rhino`, `Elephant`, `Snake`. 虽然我们人知道他们都继承于 `Animal`, 但是 typescript 并不会去做这样的推导，他只会看他已知的这几个类型，发现并不能找到一个类型能兼容其他的, 所以 `zoo` 就会被推导成兼容这3种类型的联合类型的数组: `(Rhino | Elephant | Snake)[]`.

如果我们改一下上例，

```
let zoo = [new Rhino(), new Elephant(), new Snake(), new Animal];
```
这是 `zoo` 就会被推导成 `Animal[]` 了。

# 类型兼容性
## 对象和类的兼容性
一般来说是“鸭式辨型法”, 但是也有例外：[对象字面量的额外属性检查](#额外的属性检查), [继承于有非 public 成员的类的接口](#继承于类的接口)。

类的兼容性方面还有2个特别的之处:

1. 比较两个类类型的对象时，只有实例的成员会被比较。 静态成员和构造函数不在比较的范围内。

  ```
  class Animal {
      static x: string;
      feet: number;
      constructor(name: string, numFeet: number) { }
      static method() { }
  }

  class Size {
      static y: string;
      feet: number;
      constructor(numFeet: number) { }
  }

  let a: Animal;
  let s: Size;

  a = s;  //OK
  s = a;  //OK
  ```

2. 类的非 public 成员会导致 ts 的类型系统从 “鸭式辨型法” 升级为 `nominal typing`.

如下例:
```
class Animal {
  private feet: number;
}

class Size {
  feet: number;
}

let a: Animal;
let s: Size;

a = s;
s = a;
```

## 函数
参数少了没关系, 返回值多了没关系。仔细想一下, 也很好理解。

```
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

```
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error because x() lacks a location property
```

### 函数参数双向协变 和 可选参数及剩余参数, 函数重载
函数参数双向协变: 看例子似乎就是强制类型装换，这样做很危险。
可选参数及剩余参数: 完全未明其义。
函数重载: 没有什么特别之处

## 枚举
* 数字枚举和 `number` 互相兼容。
* 非数字枚举的兼容性很奇怪，请自行测试，此处不一一列举。
* 不同枚举不兼容。

```
enum AnimalType { Dog, Elephant, Fish }

enum Color { Red, Blue, Green };

let a: number = AnimalType.Dog
let b: AnimalType = 3
let c: Color = AnimalType.Dog // 不兼容
```

## 泛型类型

泛型类型比较时, 会把所有泛型参数当成 any 去比较

```
let identity = function<T>(x: T): T { }
let reverse = function<U>(y: U): U { }
identity = reverse; // Okay because (x: any)=>any matches (y: any)=>any
```

# 高级类型
## 交叉类型

表示包含多种类型所有属性的类型，多用于对象的合并:

```
class Animal {
    constructor(public name: string) { }
}

class Fly {
    constructor(public fly: string) { }
}

let bird: Animal & Fly = Object.assign(new Animal('bird'), new Fly('can fly'))
```

上例中 `bird` 包含 Animal 和 Fly 类的所有属性, 其类型可表示为 Animal & Fly。

## 联合类型
联合类型表示一个或者的关系，比如 `string | number`, 表示这个变量可以是 `string`, 也可以是 `number`.

```
function print(text: string | number) {
  console.log(text);
}

print(1);       // OK
print('2');     // OK
print(false);   // error
```

## 类型保护

```
class Bird {
  fly() {}
  layEggs() {}
}

class Fish {
  swim() {}
  layEggs() {}
}

function getPet(): Fish | Bird {
  return Math.random() > 0.5 ? new Fish() : new Bird()
}

let pet = getPet() // getPet()的返回值类型是`Bird | Fish`
pet.layEggs() // 允许, Bird 类 和 Fish 类都有此方法
pet.swim() // 报错, pet 的类型不能确定为 Fish
```

要解决这个问题:
1. 可以通过类型强转（即所谓的断言）后, 然后检查函数是否存在来判断类型
    ```
    (pet as Fish).fly && (pet as Fish).fly()
    ```
  ts 对这种方案处理的不太好, 所以需要2次转. 看来也是不建议使用。

2. 使用 `instanceof` 关键字
    ```
    if (pet instanceof Fish) {
      pet.swim()
    }
    ```
  ts 对这个方式支持不错

3. ts 还提供了一个**清奇**的方案，即本节的主角**类型保护**
    ```
    function isFish(pet: Fish | Bird): pet is Fish {
      return (<Fish>pet).swim !== undefined;
    }

    isFish(pet) && pet.swim()
    ```
    WTF!? 好吧，ts 提供了这么一个特殊的返回值后, 被这种返回值的函数调用过后，参数的类型就确定下来了。先知道一下吧，返回值的类型格式为 `parameterName is Type`。

    > 真的很惊喜和意外，也很受惊。😂

## typeof 类型保护
使用 typeof 时, 当期作用于 `number`, `string`, `boolean` 以及 `symbol` 时, ts 对其做了类型保护, 即类型会变得明确。

```
function getVariable() {
  let r = Math.random()

  if (r < 0.25) {
    return 1
  } else if (r >= 0.25 || r < 0.5) {
    return 'i am string'
  } else if (r >= 0.5 || r < 0.75) {
    return true
  } else () {
    return Symbol()
  }
}

let v = getVariable() // type: number | string | boolean | symbol

if (typeof v  === 'number') {
  // v is number
} else if (typeof v === 'string') {
  // v is string
} else if (typeof v === 'boolean') {
  // v is boolean
} else (typeof v === 'symbol') {
  // v is symbol
}
```

## 用 ! 去除 null 和 undefined 类型
这是开启 strictNullChecks 选项后的一个技巧。一般来说想要去除 null 或者 undefined 类型，用简单的判断就够了, 比如：

```
function test(name: string | null) {
    return name === null
        ? ''
        : name.charAt(0)
}
```

一些特殊情况下，你可能需要强制去去除 null 和 undefined 类型，比如：

```
function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // ok
  }
  name = name || "Bob";
  return postfix("great");
}
```

## 类型别名

```
  type NameResolver = () => string;
  type Container<T> = { value: T };                 // 和泛型一起使用
  type LinkedList<T> = T & { next: LinkedList<T> }; // 和交叉类型, 泛型一起使用
  type Yikes = Array<Yikes>; // error               // 不可以出现在右侧
```
类型别名不创造一个新的类型，它仅仅是别名。所以你无法 extends 或者 implements 它。

### 何时用类型别名？何时用接口？
接口会创建实实在在的类型。如果你需要表示对象，一般建议使用interface。别名更适合用于表示`联合类型`, `交叉类型`等书写起来冗长的类型。

```
// good
interface CoordinateA { x: number; y: number}
function printA(c: CoordinateA) {
    console.log(c.x, c.y)
}
let a: CoordinateA = { x: 1, y: 2 }
printA(a)

// bad
type CoordinateB = { x: number; y: number}
function printB(c: CoordinateB) {
    console.log(c.x, c.y)
}
let b: CoordinateB = {x: 3, y: 4}
printB(b)

// 合适
type MyType = string | number | boolean | symbol
function demoFunction(data: MyType): MyType { return MyType } // 简洁
```

## 字符串和数字字面量类型
字符串字面量和数字字面量也是类型。

```
function print(val: 'cat') {}
print('cat') // ok
print('dog') // error

function print1(val: 1) {}
print1(1) // ok
print1(2) // error
```
这个例子有点神经病了，不过只是展示。这种类型更多的是像下例这样使用:

```
// 工厂方法
function creator(type: 'dog' | 'cat' | 'horse' ): Animal {
  switch(type) {
    case: 'dog':
      return new Dog();
    case: 'cat':
      return new Cat();
    case: 'horse':
      return new Horse();
  }
}

let animal = creator('dog')     // OK
let animal1 = creator('snake')  // error
```

## 可辨识联合
所有类型的共有属性，可以在他们的联合类型上访问:

```
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle; // 3个类型共有 kind 类型

function area(s: Shape) {
    switch (s.kind) {                     // Shape 类型就有 kind 类型
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

## 完整性检查
属于开发技巧, 详见:
https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/Advanced%20Types.html

## 多态的 this 类型
`this` 即是变量，又是类型，其具有多态性。this 返回的是对象本身，这种多态性很好理解。

```
class BasicCalculator {
    public constructor(protected value: number = 0) { }
    public currentValue(): number {
        return this.value;
    }
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
    public multiply(operand: number): this {
        this.value *= operand;
        return this;
    }
}

class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }
    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
}

let v = new ScientificCalculator(2)
  .multiply(5)
  .sin()              // 体现多态
  .add(1)
  .currentValue();
```

## 索引类型

```
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};

let strings: string[] = pluck(person, ['name']); // ok, string[]
```
例子有点复杂, 这几个特别的操作符:

* keyof: 索引类型查询操作符。 对于任何类型T，`keyof T` 的结果为T上已知的公共属性名的联合。即:

  ```
  let personProps: keyof Person; // 'name' | 'age'
  ```

* T[K]: 索引访问操作符，通过 K 类型能访问到的值。

这 2 点都还好理解, 我个人一直觉得奇怪的就是 `K extends keyof T` 这个表达式了。后来想了一下，我们可以分开这样来理解:

* `K extends T`, 是泛型约束, 表示 K 是 T 的一个子类或者是 T 类型本身。
* `keyof T` 是一个联合类型, 联合类型似乎不太好继承, 所以 `K extends keyof T` 里的 `K` 类型就等价于 `keyof T` 这个类型本身。

`K extends keyof T` 这种写法相当于给 keyof T 搞了一个别名 K, 且同时能够强调参数 name 的这个 `keyof T` 类型和返回值里这个 `keyof T` 类型是一致的，有助于类型推断。这样说有点抽象，我们把上例换个写法:

```
interface Person {
    name: string;
    age: number;
}

function pluck<T>(o: T, names: Array<keyof T>): T[keyof T][] {
  return names.map(n => o[n]);
}

let person: Person = {
    name: 'Jarid',
    age: 35
};

let strings: Array<string | number> = pluck(person, ['name']); // 类型无法推断
```
注意最后一行, `strings` 无法被推断成 `strings[]` 类型了。
这就是 `function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][]` 这个声明厉害的地方了，表达了多重的信息，而且只要一行代码。

## 映射类型
把 for...in 语法用用到了类型声明, 一般用于改变属性的一些特性:

```
type Keys = 'option1' | 'option2';
type Flags = { [K in Keys]: boolean };
```

等价于

```
type Flags = {
    option1: boolean;
    option2: boolean;
}
```

映射类型来创造一些类型:

```
type Nullable<T> = { [P in keyof T]: T[P] | null }
type Partial<T> = { [P in keyof T]?: T[P] }

interface Name {
  firstName: string;
  lastName: string;
}

type NullableName = Nullable<Name>
type PartialName = Partial<Name>
```

那么相当于:

```
type NullableName = {
  firstName: string | null;
  lastName: string | null;
} // 当然开启 strictNullChecks 选项后，这个声明才有意义

type PartialName = {
  firstName?: string;
  lastName?: string;
}
```

一下这4个映射类型都属于 ts 标准库, 可以直接被使用。

```
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}

type Partial<T> = {
    [P in keyof T]?: T[P];
}

type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}

type Record<K extends string, T> = {
    [P in K]: T;
}
```

唯一需要说明的是字符串字面量类型和他们的联合都属于 `K extends string` 类型。

```
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>
```
等价于
```
type ThreeStringProps = {
  prop1: string;
  prop2: string;
  prop3: string;
}
```

一个复杂的例子:

```
type Proxy<T> = {
    get(): T;
    set(value: T): void;
}
type Proxify<T> = {
    [P in keyof T]: Proxy<T[P]>;
}

function proxify<T>(o: T): Proxify<T> {
    let result = {} as Proxify<T>

    for (let prop in o) {
        Object.defineProperty(result, `_${prop}`, {
            writable: true,
            enumerable: false,
            value: o[prop]
        });
        result[prop] = {
            get() { return result[`_${prop}`] },
            set(value) { result[`_${prop}`] = value }
        }
    }

    return result
}

let proxyProps = proxify({
    a: 1,
    b: 2
});

alert(proxyProps.a.get())
proxyProps.a.set(9)
alert(proxyProps.a.get())

function unproxify<T>(t: Proxify<T>): T {
    let result = {} as T;
    for (const k in t) {
        result[k] = t[k].get();
    }
    return result;
}

let originalProps = unproxify(proxyProps);

console.log(originalProps)
```

看着这些泛型，头晕了么。😂

# 模块
和 es6 的模块系统区别不大。`export` and `import` 你懂的。

导出:
```
// 导出变量
export let a = 1;

// 导出类
export class Test { }

// 导出函数
export function Test () {}

// 导出默认函数, 函数名可以省略
export default function () {}

// 导出默认类
export default class {}

// 导出默认变量，类名可以省略
export default {
  a: 1,
  b: 2
}

// 导出另一个名字
export { Test as Test1, a as a1 }

// 重新导出
export { A, B1 as B } from "./ZipCodeValidator";

// 重新导出所有
export * from "./StringValidator";
```

导入:
```
// 导入默认和个别的导出变量
import obj, { a, Test } from 'export.ts';

// 导入所有导出变量
import * as validator from "./ZipCodeValidator";

// 全局导入
import "./my-module.js";
```

```
declare let $: JQuery;
export default $;
```

## 生成模块代码
根据编译选项的不同，ts 代码可以被编译成各种不同的模块，详见我之前写的一篇[文章](2018-3-24-typescript-compiler-options.md#module)。

> 有一点值得注意，ts 编译出来的所谓的 UMD 模块, [并不支持全局变量](https://github.com/Microsoft/TypeScript/issues/8436)，不能再浏览器中直接使用。

## 模块的动态加载
使用

```
declare function require(moduleName: string): any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
    let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator");
    let validator = new ZipCodeValidator();
    if (validator.isAcceptable("...")) { /* ... */ }
}
```

## 使用其它的JavaScript库
需要这个库的定义文件`.d.ts`, 类似于 C++ 里的头文件。

例子:
```
// node.d.ts
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export let sep: string;
}
```

通过 `/// <reference path="[type file path]">` 语法，可以引入 `.d.ts` 文件

```
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");
```

> 现在常用的库，都已经在 npm 包里带了 type 定义文件，所以 ``/// <reference>` 语法用的很少了。

## 外部模块声明简写

```
// declarations.d.ts

declare module "hot-new-module";
```

使用这个模块:

```
import x, {y} from "hot-new-module";
x(y);
```

这样写所有的导出类型都是 any。不过这可能是最常用的方案了，一般你可能都懒得去为一个第三方库写 type 定义文件, 真的挺难写的。

不过还有更简单的方法：

```
let x = require('hot-new-module')
```
对于没有 type 定义的库，直接通过 `require` 引入, 则 x 即为 any 类型，不需要专门去写个 definition.


## 加载非 js 内容

```
declare module "*!text" {
    const content: string;
    export default content;
}

declare module "json!*" {
    const value: any;
    export default value;
}
```

```
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```

## UMD 模块

UMD 模块的声明形式:

```
export function isPrime(x: number): boolean; // 这个库可以被当做模块使用
export as namespace mathLib;                 // 这个库可以在非模块脚本里作为全局变量来访问
```

## 创建模块结构指导

> best practice for 模块的类型导出: '尽可能地在顶层导出', 详见：[尽可能地在顶层导出](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/Modules.html#尽可能地在顶层导出)

# 命名空间

文件（模块）可以定义多个命名空间，来避免面重名。

```
namespace Module1Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }

  export interface ArrayValidator {
    isAcceptable(s: string): boolean;
  }
}

namespace Module2Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }

  export interface ArrayValidator {
    isAcceptable(s: string): boolean;
  }
}

let validator1: Module1Validation.StringValidator = null
let validator2: Module2Validation.StringValidator = null

import stringValidator = Module1Validation.StringValidator;
                  // 这个清奇的语法用于给 interface 里的内容取别名
```

多个同名的命令空间(可分散于不同的文件中)会被自动 merge。

```
// 假设你安装了 node 的 types: @types/node
namespace NodeJS {
  export let a = 100
}

class Test implements NodeJS.InspectOptions {}
console.log(NodeJS.a)
```

`NodeJS.InspectOptions` 的定义来自于 `node_modules/@types/node/index.d.ts`

## 命名空间来描述第三方全局 js 库
前面我们提到了用 module 去描述 js 写的第三方库。但是 module 描述的是模块化的库。对于一些老旧的库，他们不是模块化的，而是直接使用全局变量。此时, 可以通过外部的命名空间来声明:

比如有一个这样的 js 库：
```
// myLib.js
(function (window) {
  var timeout = 20
  var version = '1.0.1'

	function Cat(age) {
    this.age = age
	}
	Cat.prototype.purr = function () {
    console.log('purr')
	}

	window.myLib = {
    Cat: Cat,
    timeout: timeout,
    version: version
  }
})(window)
```

现在我想在 typescript 脚本里使用这个库的全局变量 `myLib`, 于是我需要用 `namesapce` 来做一个声明：

```
// lib.d.ts
declare namespace myLib { // 外部命名空间
  class Cat {
    constructor(age: number);

    purr(): void;
    age: number;
  }

  let timeout: number;
  const version: string;
}
```

引入这个声明文件，就可以用起来了:

```
/// <reference path="./test.d.ts"/>
let lib = new myLib.Cat(1) // OK
console.log(myLib.timeout) // OK
myLib.version = '1.0.1'    // error, version is const
```

在**无导入导出的 ts 脚本**（即外部模块）中, 顶层的命令空间，都是外部命名空间。比如上一节例子中的，`Module1Validation`, `Module2Validation`

# 全局模块和文件模块
https://basarat.gitbooks.io/typescript/content/docs/project/modules.html

# 命名空间和模块对比

* **命名空间**本质上是一个**全局的**，普通的带有名字的JavaScript对象，它能够跨文件合并，但会造成全局污染。
* **模块**则是更现代化的代码组织方式, 需要通过导入导出来使用, 不会有全局污染。是更推荐的方式。

# 模块解析
具体的解析过程，这里就不记录了。如果遇到模块解析的问题，打开 `traceResolution` 标记分析一下就行了。相关的选项如下:

* [traceResolution](2018-3-24-typescript-compiler-options.md#traceResolution)
* [baseUrl](2018-3-24-typescript-compiler-options.md#baseUrl)
* [paths](2018-3-24-typescript-compiler-options.md#paths)
* [rootDirs](2018-3-24-typescript-compiler-options.md#rootdirs)

# 声明合并
Typescript 的实体大致分为3中，即：`Namespace`，`Type` 和 `Value`.
当我们做一个声明的时候，有时候我们同时创建了多个实体。最为典型的就是 `class`。下面的这个声明不仅创建了一个 Animal 类型（Type），同时也是创建了值, 即一个构造函数（Value）。

```
class Animal {
}

let animal: Animal;
console.log(Animal);
```

我们来看一下完整的声明和对应实体的关系:

| Declaration Type | Namespace | Type | Value |
|------------------|:---------:|:----:|:-----:|
| Namespace        |     X     |      |   X   |
| Class            |           |   X  |   X   |
| Enum             |           |   X  |   X   |
| Interface        |           |   X  |       |
| Type Alias       |           |   X  |       |
| Function         |           |      |   X   |
| Variable         |           |      |   X   |

## 接口合并
相同名称的接口会被合并，合并原则如下：

* 变量名称不一样的，直接合并到一起，一样的，则必须类型相同。

    ```
    // 合体会失败
    interface Box {
        height: number;
    }

    interface Box {
        height: string;
    }
    ```

    ```
    // 合体会成功
    interface Box {
        width: number;
        length: number;
    }

    interface Box {
        height: string;
        length: number;
    }

    合体后:

    interface Box {
        width: number;
        height: string;
        length: number;
    }
    ```

* 函数名称不一样的，直接合并到一起，一样的，合并成重载.

    ```
    interface Box {
        test(s: string): number;
    }

    interface Box {
        test(n: number): string;
    }

    合体后:

    interface Box {
        test(n: number): string;    // 注意后面定义的，出现在重载类型的前面。
        test(s: string): number;
    }
    ```
> 命名空间的合并原则和**接口合并**类似。

## 命名空间与其他类型合并

* 和类合并，以表示内部类

    ```
    class Album {
        label: Album.AlbumLabel;
    }
    namespace Album {
        export class AlbumLabel { }
    }
    ```

* 和函数合并，在函数上扩展属性

    ```
    function buildLabel(name: string): string {
        return buildLabel.prefix + name + buildLabel.suffix;
    }

    namespace buildLabel {
        export let suffix = "";
        export let prefix = "Hello, ";
    }

    alert(buildLabel("Sam Smith"));
    alert(buildLabel.prefix);
    ```
* 和枚举合并，扩展枚举

    ```
    enum Color {
        red = 1,
        green = 2,
        blue = 4
    }

    namespace Color {
        export let yellow = 5
    }

    function printColor(color: Color) {
        alert(color);
    }

    printColor(Color.yellow)
    ```

## 扩展模块
假设有一个第三方模块 module，我们不便去修改他的代码

```
// module.ts
export class A {}
```

但是我们想给这个导出函数 A 扩展一个 test 方法，可以这样做:

```
// test.ts
import { A } from "./module";
declare module "./module" {
    interface A {
        test(): void
    }
}
A.prototype.test = function () {
}
```

最后可以这么使用:
```
// consumer.ts
import { A } from "./module";
import './test';

let a: A = new A();
a.test()
```

> 麻烦、令人困惑，还是少这么用吧。😂

## 扩展全局变量
在全局模块里这是很容易的。

```
   interface Array<T> {
     test(): void
   }

   Array.prototype.test = function() {}

   new Array().test()
```

如果你想在文件模块里给全局的类型作用于添加声明，则需要使用 declare global 语法:

```
    export function test() {}   // export 让这个文件变成了一个文件模块

    declare global {            // 使用 declare global
        interface Array<T> {
          test(): void
        }
    }

    Array.prototype.test = function() {}
    new Array().test()
```

`declare global` 语法让文件模块里的 interface 能像全局模块里的一样去和其他全局的定义进行 merge.


# 三斜线指令

```
/// <reference path="..." />    // 引入的额外的文件
/// <reference types="..." />   // 引入额外申明, 一般只在自己写
                                // definition 文件的时候才主动使用
```
这2个语法 typescript 2.0 后, 使用的越来越少了。直接通过 npm 安装申明文件后，typescript 就可以正确的引入类型文件了，一般情况下已经不需要这样语法了。

```
npm install --save @types/lodash

import * as _ from "lodash"; // 不在需要 reference 了。
_.padStart("Hello TypeScript!", 20, " ");
```

其他三斜线指令遇到的话，请参考 [官方文档](http://www.typescriptlang.org/docs/handbook/triple-slash-directives.html)。

# 混入
运行时再去实现类的成员方法的技术。
[官方文档](http://www.typescriptlang.org/docs/handbook/mixins.html), [中文版](https://zhongsp.gitbooks.io/typescript-handbook/doc/handbook/Mixins.html)

# Decorators
面向切片编程(AOP)的核心, 装饰器, 虽然在 js 上是一个实验性的方案，但在 react.js 的 HOC 中被广泛应用。
Typescript 中，通过 [experimentalDecorators](https://www.njleonzhang.com/2018/03/24/typescript-compiler-options.html#experimentaldecorators) 选项启用。

分为多种：
* 类装饰器
* 函数装饰器
* 访问器装饰器
* 属性装饰器
* 参数装饰器

所有的装饰器实现上都是一个函数。

## 类装饰器
一个`类装饰器`的类型可表示为其类型可表示如下:

```
/**
 *  Ctor: 类的构造函数
 */
(Ctor: new(args: any[]) => any) => void | (new(args: any[]) => any)
```
如果`类装饰器`返回一个类，它会被用来替换原来的类。

### 没有返回的类装饰器
```
function addAge(constructor: Function) {
  constructor.prototype.age = 18;
}
​
@addAge
class Hello {
  name: string;
  age: number;
  constructor() {
    this.name = 'leon';
  }
}
​
let hello = new Hello();
​
console.log(hello.age);
```

可以认为, `类装饰器`使用的语法相当于:

```
addAge(Hello)
```

### 返回类的类装饰器

```
interface IHello {
    name: string;
    age: number;
    sayHello();
}

type HelloConstructor = new(name: string, age: number) => IHello;

// 下面2种写法也可以:

// type HelloConstructor = { new(name: string, age: number): IHello };

// interface HelloConstructor {
//     new(name: string, age: number): IHello
// }

function Test(Ctor: HelloConstructor): HelloConstructor {
    return class Hoc extends Ctor {                 // 这个类装饰器，返回了一个类
        constructor(name: string, age: number) {
            super(name, age);
            this.age = age;
        }

        sayHello() {
            console.log(`hello, ${this.name}!`);
        }
   }
}
​
@Test
class Hello implements IHello{
  constructor(public name: string, public age: number) {}

  sayHello() {}
}
​
let hello = new Hello('leon', 18);
console.log(hello.age, hello.name); // 18 "leon"
hello.sayHello();                   // hello, leon!
```

此处的 `@Test`就相当于。

```
const Hello = Test(Hello)
```

实际上我们把 `Hello` 类已经替换掉了，不过我们保持了接口的一致性。所以我们添加的这个装饰器不影响代码的兼容性。

> 如果读者熟悉 react Hoc 的话，应该已经发现 react Hoc 的原理和本例类似。

## 方法装饰器
`方法装饰器`的类型可表示为:

```
/**
 *  target: 对于静态成员函数来说是类的构造函数，对于实例成员函数是类的原型对象
 *  propertyKey: 成员函数的名字
 *  descriptor: 成员的属性描述符
*/
(target: any, propertyKey: string, descriptor: PropertyDescriptor) => PropertyDescriptor | void
```

如果`方法装饰器`返回一个值，它会被用作该方法的`属性描述符`。

栗子:

```
function enumerable(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = false;
};

function hello(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log(target, `hello ${propertyKey}`);
};

class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable
    greet() {
        return "Hello, " + this.greeting;
    }

    @hello
    static test() {
    }
}

console.log(Object.keys(Greeter.prototype))   // [], greet is not enumerable
```

此例中, greet 为实例方法, 所以 `enumerable` 的 `target` 参数为类的原型对象，即 `Greeter.prototype`, 而 `test` 是一个静态方法，所以 `hello` 的 `target` 为构造函数，即 `Greeter`。

### 访问装饰器
`访问装饰器`的类型表示、用法和 `方法装饰器` 一模一样。实质上似乎也没有啥区别。。。。

```
function configurable(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.configurable = false;
};

class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}

console.log(Object.getOwnPropertyDescriptor(Point.prototype, 'x')) // { configurable: false, ... }
```

### 属性装饰器
`属性装饰器`的类型可表示为:

```
/**
 *  target: 对于静态成员函数来说是类的构造函数，对于实例成员函数是类的原型对象
 *  propertyKey: 成员函数的名字
 */
(target: any, name: propertyKey) => void | any
```

> 如果有返回值，那么似乎一定要是一个属性描述符，但是行为颇为[古怪](https://github.com/zhongsp/TypeScript/pull/226)。

和`方法装饰器`相比，`属性装饰器` 的参数少了一个 `属性描述符`。为什么呢？因为`没有`。。。
js 中成员属性和成员方法不同，成员方法是挂在类的 prototype 对象上的, 而成员属性是在类的实例上的属性。类型 `prototype` 在类声明的时候就存在了，而类的实例是每次 `new` 之后才能存在的。装饰器都是在类创建的时候执行的，所以`属性装饰器`的函数参数里并不能有`属性描述符`, 因为`没有`。

```
function test(target: any, propertyKey: string) {
    console.log(target, propertyKey);
}

class Greeter {
    @test
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }
}
```

[官网中](https://www.typescriptlang.org/docs/handbook/decorators.html#Property%20Decorators)列举的 `属性装饰器` 的使用实例是配合 [reflect-metadata](https://github.com/rbuckton/reflect-metadata) 来为属性添加元数据，这里就不介绍了。

### 参数装饰器
`参数装饰器`的类型可表示为:

```
/**
 *  target: 对于静态成员函数来说是类的构造函数，对于实例成员函数是类的原型对象
 *  propertyKey: 成员函数的名字
 *  index: 参数在函数参数列表中的索引
*/
(target: any, name: propertyKey, parameterIndex: number) => void | any // 如果有返回值，会被忽略
```

示例:
```
function require(target: any, propertyKey: string, parameterIndex: number) {
    console.log(target, propertyKey, parameterIndex)
}

class Greeter {
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }

    greet(@require name: string) {
        return "Hello " + name + ", " + this.greeting;
    }
}
```

[官网示例](https://www.typescriptlang.org/docs/handbook/decorators.html#Parameter%20Decorators)里展示了如何使用`参数装饰器`和`函数装饰器`去做参数的验证, 此处不深入介绍。

# jsx
[官方文档](http://www.typescriptlang.org/docs/handbook/jsx.html), [中文版](https://zhongsp.gitbooks.io/typescript-handbook/doc/handbook/JSX.html)

# 如何写声明文件
[官方文档](http://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html), [中文版](https://zhongsp.gitbooks.io/typescript-handbook/doc/handbook/declaration%20files/Introduction.html)

# javascript 文件里的类型检查
可以通过加 JSDoc 注释的方式给 javascript 文件的变量指定类型。
[官方文档](http://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html), [中文版](https://zhongsp.gitbooks.io/typescript-handbook/doc/handbook/Type%20Checking%20JavaScript%20Files.html)
