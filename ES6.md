## ES6模块化

### 1. Node.js 中如何实现模块化

node.js 遵循了 CommonJS 的模块化规范。其中：

- 导入其它模块使用 require() 方法
- 模块对外共享成员使用module.exports 对象

模块化的好处：大家都遵守同样的模块化规范写代码，降低了沟通的成本，极大方便了各个模块之间的相互调用，利人利己。

### 2. 前端模块化规范的分类

在 ES6 模块化规范诞生之前，JavaScript 社区已经尝试并提出了 AMD、CMD、CommonJS 等模块化规范。

- AMD 和 CMD 适用于浏览器端的 Javascript 模块化
- CommonJS 适用于服务器端的 Javascript 模块化

太多的模块化规范给开发者增加了学习的难度与开发的成本。因此，大一统的ES6 模块化规范诞生了！

### 3. 什么是 ES6 模块化规范

ES6 模块化规范是浏览器端与服务器端通用的模块化开发规范。

ES6 模块化规范中定义：

- 每个 js 文件都是一个独立的模块
- 导入其它模块成员使用 import 关键字
- 向外共享模块成员使用 export 关键字

### 4. 在 node.js 中体验 ES6 模块化

Node.js 中默认仅支持 CommonJS 模块化规范，若想基于 node.js 体验与学习 ES6 的模块化语法，可以按照
如下两个步骤进行配置：

1. 确保安装了 v14.15.1 或更高版本的 node.js
2. 在 package.json 的根节点中添加 "type": "module" 节点

### 5. ES6 模块化的基本语法

ES6 的模块化主要包含如下 3 种用法：

1. 默认导出与默认导入
2. 按需导出与按需导入
3. 直接导入并执行模块中的代码

#### 5.1 默认导出

默认导出的语法： export default 默认导出的成员

```javascript
//当前模块为m1.js
let n1 = 10 //定义模块私有成员n1
let n2 = 20 //定义模块私有成员n2 (外界访问不到n2，因为它没有被共享出去)
function show(){} //定义模块私有方法show

export default { //使用export default默认导出，向外共享n1和show两个成员
    n1,
    show
}
```

默认导入的语法： import 接收名称 from '模块标识符'

```javascript
//从m1.js模块中导入export default向外共享的成员
//并使用m1成员进行接受
import m1 from './m1.js'

console.log(m1) // {n1: 10, show: [Function: show]}
```

每个模块中，只允许使用唯一的一次 export default，否则会报错！

默认导入时的接收名称可以任意名称，只要是合法的成员名称即可：

#### 5.2 按需导出

按需导出的语法： export 按需导出的成员

```javascript
//当前模块为m2.js
//向外按需导出变量s1
export let s1 = 'aaa'
//向外按需导出变量s2
export let s2 = 'ccc'
//向外按需导出方法say
export function say(){}
```

按需导入的语法： import { s1 } from '模块标识符'

```javascript
//导入模块成员
import {s1, s2, say} from './m2.js'

console.log(s1) //aaa
console.log(s2) //ccc
console.log(say) //[Function: say]
```

按需导出与按需导入的注意事项

1. 每个模块中可以使用多次按需导出
2. 按需导入的成员名称必须和按需导出的名称保持一致
3. 按需导入时，可以使用as 关键字进行重命名
4.  按需导入可以和默认导入一起使用

#### 5.3 直接导入并执行模块中的代码

如果只想单纯地执行某个模块中的代码，并不需要得到模块中向外共享的成员。此时，可以直接导入并执行模块代码，示例代码如下：

```javascript
//当前模块为m3.js
//执行一个for循环
for(let i=0; i<3; i++){
    console.log(i)
}
```

```javascript
//直接导入并执行模块代码，不需要得到向外共享的成员
import './m3.js'
```

