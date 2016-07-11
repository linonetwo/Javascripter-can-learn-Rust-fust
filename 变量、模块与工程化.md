
### 为什么要介绍 Rust

作为 ECMAScript 使用者，我们常常发现企业招聘要求我们掌握一门非前端语言，例如 Python、Java、C++、Julia 等。这些语言各有自己的侧重点，但我认为 Julia 和 Rust 是最容易从 ECMAScript 迁移的高性能语言了，他们在有很高的执行速度的同时，又能看到许多我们已熟悉的 ES6 的影子。不过 Rust 比起 Julia 有一个好，就是从它你可以很快地再迁移到 C++，简直一箭双雕一石二鸟一举两得一夫双妻。

如果你和我一样，是从了解知识之间的相似性而不是深挖一个知识来获得学习的快感，那么就请你跟着我一起迁移知识到 Rust 吧。


### let 和 const

##### ES6

>  `let foo = 8964;` ——前端成语

在 ECMASCript 里，我们用 let 来声明一个变量，这个变量只会在一对` {} `里有效，比如在古代，我们有时候干活干到深夜，双眼无神，就会写出:

```javascript
function varTest() {
  var x = 'girl friend';
  if (true) {
    var x = 'boy';  // now you don't have girlfriend anymore
    x = 'gay';
    console.log(x);  // 'gay'
  }
  console.log(x);  // 'gay'
}
```
导致第一个变量存储的女朋友消失了，而且再也找不回来。但如果换成 let 就好很多:
```javascript
function letTest() {
  let x = 'girl friend';
  if (true) {
    let x = 'boy';  // x is not the original x, this shows the relationship between the signifier and the signified is arbitrary, 23333
    x = 'gay';
    console.log(x);  // 'gay'
  }
  console.log(x);  // 'girl friend'
}
```
这时候 if 的语句块里我们的 x 已经不再是 function 那个更大的语句块里的 x 了，你会发现 babel 会把 `let x = 'boy'` 转换成 `var _x = 'boy'` 从而保证不覆盖之前的变量。

##### Rust

不过这边 `let x = 5;` 与 ES6 不同的是，它的意思其实是 ES6 中的 const …… 也就是说如果你接着尝试 `x = 6;` 就会报错。

如果你想要像 ES6 里那样的 let， 你得加上一个 mut ，也就是 mutable 的意思:
```rust
let mut x = 5; // mut x: i32
x = 10;
```
现在我们能改变 x 的值了，这才是原汁原味的 ES6 let。

那么既然 Rust 里的 let 同时干了 ES6 里 let 和 const 的活，那么， Rust 里的 const 是干嘛的呢？

const five: i32 = 5; 实际上会在编译的时候将程序中所有的 five 都替换成常量 5，就相当于你在那些地方直接写了个 5 一样。也就是说 const 干的是一个自动替换的活。

在 ES6 里有时候我们也会用到 var，比如 …… 好吧我举不出例子，据说它的速度会快一点，但我们再也不会去用它了。Rust 里也有跟 var 生命周期相似的声明方式，叫 static ，虽然它的作用域和 let 一样，但 static 会生成一块全局唯一的内存，并且这块内存会一直存在，即使你离开了定义它的块:
```Rust
let mut ccc : i32 = 6;
let mut aaa : *mut i32 = &mut ccc as *mut i32;
unsafe { // 涉及指针等概念需要用到 unsafe，我们先不解释它
    if true {
        static mut AAA: i32 = 2147483646; // 定义了 AAA
        aaa = &mut AAA as * mut i32;
    }
    let mut bbb : *mut i32 = aaa; // 通过引用继续在定义它的块外使用 AAA，它不会被回收
    *aaa  = *aaa + 1; // 给 AAA 加一
    let ddd: i64 = *bbb as i64 + 1; // 给 AAA 加一
    println!("Hello, world! {}", ddd);
}
```
当然这个例子仅作示意，static 有它自己的具体用法，和古代的 var 绝对不一样。

##### TS

你可能看到我在上面写了 `ccc : i32 = 6` 等记号。如果你用过 flow 或 [TypeScript](https://www.typescriptlang.org/docs/tutorial.html) ，你会发现它们都喜欢在变量 x 的后面加上一个冒号，冒号后面是一个类型，比如 `function foo(person: string) { return 'silly ' + person; }` 中的 `person: string` 的意思就是，你给这个函数传入的参数 person 一定得是一个 string 。

##### Rust
在 Rust 里，i 开头的 i32 是有符号整数类型，而 u 开头的 u32 则代表无符号整数，声明它们的时候都会给它们分配一定的空间来存储数字，但是 u32 因为不需要存储符号所以能存下更大的数字。可能的整数大小是 8，16，32 和 64 位。

所以我们知道了，i32 最大只能存 2147483647，首先，如果你再给它续一秒，那就是作死; 其次， i32 的数字没法赋值给 i64，它们是两个世界的数字，不能在一起。除非你深谙 64 之道，这样将它转换为 i64 :
```rust
let aaa: i32 = 2147483647;
let bbb: i64 = aaa as i64 + 1;
```
这里 `as i64` 的意思是临时地将 aaa 看做是64位整数，这样首先给它续一秒的时候就不会爆了; 其次，在将它赋值给 bbb 的时候它们就具有相同的类型 : i64 了。

要注意 `(aaa + 1) as i64` 是会爆的哈 ……

### 包管理

##### ES6
我们好像在哪里见过 as ……

是不是在 import npm 包的时候？
```javascript
import { v1 as neo4j } from 'neo4j-driver';
```
我们知道 ES6 里 export 一个东西有两种方式，第一种是 `export default function foo() {}` ，这时我们导入包是用 `import foo from './foo';` ; 而如果你用第二种方式  `export function v1() {}; export function v2() {}` ，那么就得用 `import { v1 } from './foo';` 来导入啦。但是我们希望导入的东西能有一个对我们有意义的名字，这时候我们用 `import { v1 as foo } from './foo';` 。



### 你写 ES， 你肯定用过 package.json

##### Rust

在 Rust 里，程序不像 ES 一样是从一个 js 文件的最上面一行一行往最下面执行，它有一个程序入口点，整个 Rust 程序都由它启动:
```rust
fn main() {
    let x = 5;
}
```
你可以想象这个 main 函数就像我们放在 ./bin/ 文件夹里的[ js 文件](https://github.com/linonetwo/poselevel/blob/master/bin/server.js)。

我们不是一般会通过 package.json [这样](https://github.com/linonetwo/poselevel/blob/master/package.json#L43)调用嘛:

package.json
```json
{
  "betterScripts": {
    "start-prod": {
      "command": "node ./bin/server.js"
    }
  }
}
```

./bin/server.js
```javascript
#!/usr/bin/env node
require('../server.babel'); // babel registration (runtime transpilation for node)
var WebpackIsomorphicTools = require('webpack-isomorphic-tools');
global.webpackIsomorphicTools = new WebpackIsomorphicTools(require('../webpack/webpack-isomorphic-tools'));
```
上面这个就类似于 Rust 里的 `fn main()`。

























参考:
[rust 程序设计语言 中文版 1.9.0](https://www.gitbook.com/book/kaisery/rust-book-chinese/details) 应该有部分过时的内容

rustprimer-1.1.2

[babel到底将代码转换成什么鸟样](https://github.com/lcxfs1991/blog/issues/9)
