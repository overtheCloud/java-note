### module（模块）

在 Node.js 模块系统中，每个文件都被视为一个独立的模块。 例如，假设有一个名为 `foo.js` 的文件：

```js
const circle = require('./circle.js');
console.log(`半径为 4 的圆的面积是 ${circle.area(4)}`);
```

在第一行中， `foo.js` 加载了与 `foo.js` 在同一目录中的 `circle.js` 模块。

以下是 `circle.js` 的内容：

```js
const { PI } = Math; // 变量外的大括号是解构分配，相当于 const PI = Math.PI

exports.area = (r) => PI * r ** 2;

exports.circumference = (r) => 2 * PI * r;
```

`circle.js` 模块导出了 `area()` 和 `circumference()` 函数。 通过在特殊的 `exports` 对象上指定额外的属性，可以将函数和对象添加到模块的根部。

模块内的本地变量是私有的，因为模块由 Node.js 封装在一个函数中（详见[模块封装器](http://nodejs.cn/s/rrmTGc)）。 在这个例子中，变量 `PI` 对 `circle.js` 是私有的。

可以为 `module.exports` 属性分配新的值（例如函数或对象）。

下面的例子中， `bar.js` 使用了导出 Square 类的 `square` 模块：

```js
const Square = require('./square.js');
const mySquare = new Square(2);
console.log(`mySquare 的面积是 ${mySquare.area()}`);
```

`square` 模块定义在 `square.js` 中：

```js
// 赋值给 `exports` 不会修改模块，必须使用 `module.exports`。
module.exports = class Square {
    constructor(width) {
        this.width = width;
    }

    area() {
        return this.width ** 2;
    }
};
```

模块系统在 `require('module')` 模块中实现。

### Module.exports 和 exports 的区别

每一个 node.js 执行文件，都自动创建一个module对象，同时，module 对象会创建一个叫 exports 的属性，初始化的值是 {}

```js
exports = module.exports = {};
```

- **【重要】exports 其实是 module.exports 的一个引用。**
- module.exports 初始值为一个空对象 {}，所以 exports 初始值也是 {}
- require 引用模块后，返回的是 module.exports 而不是 exports!!!!!
- exports.xxx 相当于在导出对象上挂属性，该属性对调用模块直接可见
- exports = 相当于给 exports 对象重新赋值，调用模块不能访问 exports 对象及其属性
- 如果此模块是一个类，就应该直接赋值 module.exports，这样调用者就是一个类构造器，可以直接 new 实例。

**示例**

```js
// foo.js
exports.a = function(){
    console.log('a')
}

module.exports = {a: 2}
exports.a = 1

// test.js
var x = require('./foo');

console.log(x.a)

// 执行结果 2
// module.exports 被修改后 exports 就失效了，可以通过 `exports = module.exports` 的方式使其恢复原来的特点
```

- [官方文档API](http://nodejs.cn/api/modules.html)