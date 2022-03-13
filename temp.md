# temp

## 模块

### ES6

循环加载：ES6模块在编译时输出接口，但使用该接口时发现未定义会报错。

```JavaScript
// a.mjs
import {bar} from './b';
console.log(bar);
export let foo = 'foo';

// b.mjs
import {foo} from './a';
console.log(foo); // 报错
export let bar = 'bar';
```

- 解决方法：将接口写作函数（变量提升）

```JavaScript
// a.mjs
import {bar} from './b';
console.log(bar());
function foo() { return 'foo' }
export {foo};

// b.mjs
import {foo} from './a';
console.log(foo());
function bar() { return 'bar' }
export {bar};
```

ES6 模块加载 CommonJS 模块：`import packageMain from 'commonjs-package';` 只能整体加载（通过Node.js的内置方法`module.createRequire()`也可实现，但写法会混合ES6和CommonJS，不建议使用）

### CommonJS

- 特点：
  - CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
  - CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
    - CommonJS 加载的是一个对象，该对象只有在脚本运行完才会生成
  - CommonJS 模块的`require()`是同步加载模块（多次加载仅加载一次），ES6 模块的import命令是异步加载，有一个独立的模块依赖的解析阶段。

- 导出：将exports对象导出（默认为空对象）
  - 方法一：`exports.变量 = xx`
  - 方法二：`module.exports = 对象` 该属性指向exports对象

- 引入：`require（'文件名'）` 加载模块，返回模块导出的exports对象

- 循环加载：所有模块只会执行一次，因此当某个已执行模块被再次加载时，就只读取缓存中的值并输出，不会继续执行未执行部分。

模块的package.json文件：

- main字段：`"main": "./src/index.js"` 指定入口文件
- exports字段：
  - 子目录别名：`"exports": {"./submodule": "./src/submodule.js"}` 指定`src/submodule.js`别名为`submodule`
  - main的别名：`"exports": {".": "./main.js"}` `.`代表模块的主入口，这种写法优先级高于main字段
    - 可简写为`"exports": "./main.js"`
  - 条件加载：`"exports": {".": {"require": "./main.cjs", "default": "./main.js"}}` require指定CommonJS的入口，default指定其他情况的入口
    - 需要打开`--experimental-conditional-exports`标志

CommonJS 模块加载 ES6 模块：

```JavaScript
(async () => {
  await import('./my-app.mjs');
})();
```

## 异步迭代器

普通迭代器在调用时必须立刻返回结果，对于异步操作只能通过将value属性设为Promise对象来实现。异步迭代器解决了这一痛点。

- 异步迭代器部署在对象的`Symbol.asyncIterator`属性上
- 异步迭代器也有`next`方法，调用时返回一个Promise对象，其状态变为成功后的回调函数的参数为具有value, done属性的对象

- `for await...of`：只能在async函数内使用，用于遍历异步的Iterator接口。（也可用于同步迭代器）
  - 自动调用该对象异步迭代器的next方法，当返回的Promise对象状态变为成功时，将value属性赋值给指定变量，并进入循环体
  - 若返回的Promise对象状态变为失败，`for await...of`就会报错

## 异步生成器

`async function* gen() {}`

- 调用时返回一个异步生成器对象，对该对象调用next方法，返回一个Promise对象

### 二进制数组

- ElementType：`Int8`（8位整数）、`Uint8`（8位无符号整数）、`Int16`、`Uint16`、`Int32`、`Uint32`、`Float32`、`Float64`

- 字节序：字节序由JS运行时所在系统决定，大端/小端字节序表示高位字节处于低位字节的前侧/后侧。

#### ArrayBuffer

缓冲，代表储存二进制数据的一段内存，不能直接读写，是所有定型数组及视图引用的基本单位，可被垃圾回收机制回收。

- 创建：`new ArrayBuffer（字节数）`  在内存中分配一段连续区域，创建后不能调整大小，初始值为0
- 实例属性：`byteLength`
- 实例方法：`slice（...）`
- 静态方法：`isView（变量）` 返回布尔，表示参数是否为ArrayBuffer的视图实例（DataView和TypedArray）
- 与字符串相互转换：`TextEncoder`、`TextDecoder`

#### TypedArray

定型数组，一种视图，特定于一种ElementType并且遵循系统原生的字节序

- 定型数组一共有9种类型，以下为其构造函数名：`Int8Array`（8位整数）、`Uint8Array`（8位无符号整数）、`Uint8ClampedArray`（8位无符号整数，溢出处理不同）、`Int16Array`、`Uint16Array`、`Int32Array`、`Uint32Array`、`Float32Array`、`Float64Array`

- 创建：
  - `new TypedArray（ArrayBuffer实例，开始的字节序号，长度）`
  - `new TypedArray（长度）`
  - `new TypedArray（普通数组 / 定型数组）`
  - 传入其他类型定型数组，生成定型数组长度不变，各元素自动转换格式，缓冲字节数会自动调整

- 实例属性：
  - `length`、`byteLength`、`byteOffset` 定型数组是从底层ArrayBuffer对象的哪个字节开始的
  - `buffer` 返回整段内存区域对应的ArrayBuffer对象（只读）

- 静态属性：`BYTES_PER_ELEMENT` 每个元素的字节数

- 实例方法：
  - 具有数组的大部分实例方法，除了可能修改数组大小的方法（concat、pop、push、shift、splice、unshift）
  - `set（定型数组/普通数组[，start]）` 复制传入数组的值到指定索引处
  - `subarray（[start][，end]）` 返回子定型数组

- 静态方法：`from/of`方法：`Int8Array.from（普通数组）/ of（参数序列）`

- 溢出：上溢和下溢表示超过类型的最大/最小值。上溢和下溢不影响其他成员。上溢时舍去溢出部分

#### DataView

一种视图，专为文件和网络I/O设计，支持对缓冲数据的高度控制，相比其他视图性能略差，对缓冲内容无预设，无法迭代

- 所有DataView的API可接受一个可选的布尔值，若为true，启用小端字节序。默认为false
- 创建：`new DataView（ArrayBuffer实例 [，开始的字节序号][，字节数]）`  默认使用全部ArrayBuffer
- 实例属性：`buffer`、`byteLength`、`byteOffset`
- 实例方法：`getInt8（索引）/ setInt8（start，值）` 从指定索引的字节处开始，获取/设置一个8位整数。支持所有ElementType

### 修饰器

修饰类：

```JavaScript
@decorator
class A {}
// 等同于
class A {}
A = decorator(A) || A;
```

修饰方法：在每次方法执行前，执行修饰器函数

```JavaScript
@readonly
  name() { return `${this.first} ${this.last}` }
```

- 类的修饰器函数的`function testable(target) {}` target为被修饰的类
- 方法的修饰器函数：`function readonly(target, name, descriptor){}`
  - target为类的原型、name为装饰的属性名、descriptor为该属性的描述对象
- 若需要接受参数，可以在修饰器外再封装一层函数，该函数返回修饰器

```JavaScript
@testable(true)
class MyTestableClass {}
```

- 有多个方法修饰器时，从外到内进入，在从内向外执行

```JavaScript
function dec(id){
  console.log('evaluated', id);
  return (target, property, descriptor) => console.log('executed', id);
}

class Example {
    @dec(1)
    @dec(2)
    method(){}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```
