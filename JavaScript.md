# JavaScript

- [JavaScript](#javascript)
  - [注意事项](#注意事项)
  - [综述](#综述)
    - [JavaScript相关](#javascript相关)
    - [字符编码](#字符编码)
    - [其他](#其他)

## 注意事项

- 所有start参数都包含本身，所有end参数都不包含本身
- NaN === NaN 返回false，`Object.is(NaN, NaN)`返回true
- 伪数组：具有length属性、按索引的方式进行存储、没有真正数组的一些方法（pop、push等）
- 伪数组的length属性在调用时会重新计算，因此若把它作为循环判据，会多次计算影响效率，可将其保存为变量
- 阻止链接跳转：在链接的地址处填`javascript:void(0);`或`javascript:;`
- js代码中可不加分号，在一行开头为括号或方括号时必须在行首加上分号。在js文件开头加分号可以防止合并压缩报错
- 正则表达式的（.*？）防止一个属性从第一个双引号的左引号匹配到最后一个双引号的右引号
- ECMAScript标准的缺陷：没有模块系统、标准库较少、没有标准接口、缺乏管理系统
- 使用扩展运算符出现对象的属性冲突时，以后面的为主
- 对超出数组最大长度或对象中不存在的属性进行读取时，返回undefined
- 较高不确定性的随机数：window.crypto.getRandomValues（）
- 若有多个全局执行上下文，使用instanceof进行类型检测时可能有问题

## 综述

浏览器包含渲染引擎和JS引擎，前者解析HTML和CSS，也称内核，后者处理JS代码

JS由ECMAScript（JavaScript语法）、DOM（页面文档对象模型）、BOM（浏览器对象模型）组成

### JavaScript相关

#### 处理机制

同步异步：同步即前一个任务结束后再执行下一个任务。异步即可以同时进行多个任务

同步任务放入主线程执行栈；异步任务（回调函数）放入任务队列（消息队列）

执行机制：

- 先执行执行栈中的同步任务、当异步任务有运行结果后，将事件（回调函数）放入任务队列、执行栈中的同步任务执行完毕后读取任务队列中的事件进入执行栈，开始执行。
- 事件绑定的回调函数在事件触发后才写入任务队列、定时器在时间到后才写入任务队列

事件循环（事件轮询）：主线程不断地重复获得异步任务（事件）、执行异步任务（事件）、再获取（事件）任务、再执行

#### 进程与线程

JS是单线程运行的，但使用H5中的Web Workers可以多线程运行。浏览器都是多线程的。

firefox和老IE是单进程的，chrome和新IE是多进程的。

定时器的回调函数是在主线程执行的，alert会暂停当前主线程的执行，并暂停计时

#### 严格模式

在脚本开头加上  “use strict”； 开启严格模式，或在函数开头加上该指令，使函数在严格模式下执行，在严格模式下：

- 使用未声明变量时报错；delete未声明变量时抛出异常；用0作为前缀表示八进制字面量时抛出异常
- LHS引用时，若在全局作用域也找不到变量，会创建一个全局变量（非严格模式）/ 抛出异常（严格模式）
- 函数内部的this不允许指向window，会置为undefined。但定时器回调函数的this及在全局作用域下直接使用this不受影响
- 不允许使用with语句
- 对函数的限制：函数不能以eval或arguments作为函数名或形参名；两个命名参数不能拥有同一个名称
- eval内部创建的变量和函数无法被外部访问
- 只设置了获取函数/设置函数的访问器属性，尝试修改/读取属性会抛出异常

### 字符编码

字符串使用两种Unicode编码混合的策略：UCS-2和UTF-16。

一个码元由16个比特（2字节）组成；码点就是对应字符的编码，码点值等于Unicode编码值（Unicode编码为十六进制）

UCS-2：适用于Unicode编码在U+0000~U+FFFF的字符，一个码点对应一个码元

UTF-16：对于Unicode编码超过U+FFFF的字符，采用该策略。是UCS-2的父集，对于UCS-2不能表示的字符，每个字符会用另一个码元去选择一个增补平面，这种每个字符使用两个16位码元的策略称为代理对。

### 其他

#### 浏览器内核

内核是支撑浏览器运行的最核心的程序。内核由很多模块组成：

主线程模块：js引擎模块、html，css文档解析模块、DOM/CSS模块、布局和渲染模块...

分线程模块：定时器模块、事件响应模块、网络请求模块

其他模块略