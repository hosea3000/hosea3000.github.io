# decorator 装饰器


js 目前也有类似于 Java 和 Python 的装饰器概念, 主要写法如下

```js
@frozen
class Foo {
  @configurable(false)
  @enumerable(true)
  method() {}

  @throttle(500)
  expensiveMethod() {}
}
```

目前在 Node.js 中还无法使用这个特性, 那么他是怎么实现的呢? 关键点在于 `Object.defineProperty` 都是用这个方法实现的

```js
class Person {
  name() {
    return 'old name';
  }
}

Object.defineProperty(Person.prototype, 'name', {
  value() {
    return 'new name';
  },
  enumerable: false,
  configurable: true,
  writable: true,
}); // 相当于重新定于了Person的name方法

const p = new Person();
console.log(p.name());
```

这个方式完全重新定义了 name 方法, 如果我的需求是 在原有的方法上做一点扩展, 那该怎样实现呢?

```js
class Person {
  constructor() {
    this.age = 17;
  }
  getAge() {
    return this.age;
  }
}

function createAgeDecorator(target, name) {
  const func = target[name]; // 先保存旧的方法
  const ageDescroptor = {
    value() {
      const age = func.apply(this); // 调用旧的方法
      if (age < 18) {
        console.log('这个人还没有成年!');
      }
      return age;
    },
    enumerable: false,
    configurable: true,
    writable: true,
  };
  Object.defineProperty(target, name, ageDescroptor); // 在类上新定义方法
}

createAgeDecorator(Person.prototype, 'getAge'); // Person 类的getAge上添加装饰器

const p = new Person();
console.log(p.getAge());
```

在 Person 类的 getAge 方法上增加了装饰器, 如果年龄不满 18 岁会有提示

参考链接:
[Javascript 中的装饰器](https://aotu.io/notes/2016/10/24/decorator/index.html)

