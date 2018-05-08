
![JavaScript正则表达式学习笔记(一) - 理论基础.png](https://user-gold-cdn.xitu.io/2018/4/3/1628b0817338d1f1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 自从年前得空写了两篇文章之后就开始忙了，这一忙就是2个月😭。当时信誓旦旦说的定期写篇博客的计划也就泡汤了🤣，不过好在最近有空，顺便总结一下这两个月遇到的几个问题。第一个问题就是项目中用到了一些正则才发现之前被自己忽略的正则是时候补一补了。恰逢今天周六😎，就把自己学习JavaScript正则表达式的笔记整理成文，写了这篇关于正则表达式理论基础的文章，希望本文能对有需要的同学提供帮助。号外：本文相对基础，大神请忽略🙈。

### 一. 基本概念

1. 正则表达式是用于匹配字符串中字符组合的模式。 一种几乎可以在所有的程序设计语言里和所有的计算机平台上使用的文字处理工具。它可以用来查找（搜索）特定的信息，也可以用来查找并编辑（替换）特定的信息。

2. 核心是匹配，匹配位置或者匹配字符。

3. 在 `JavaScript` 中，正则表达式也是对象。

4. 这些模式被用于 `RegExp` 的 `exec` 和 `test` 方法, 以及 `String` 的 `match`、`replace`、`search` 和 `split` 方法。

### 二. 创建正则表达式

#### 2.1 正则表达式的创建可以有以下三种方法。

##### 2.1.1 字面量

> `/pattern/flags`

```javascript
let reg1 = /jing ke tong xue/g;
console.log(reg1); // /jing ke tong xue/g
```

##### 2.1.2 构造函数
 
> `new RegExp(pattern [, flags])`

```javascript
let reg2 = new RegExp('jing ke tong xue', 'g');
console.log(reg2); // /jing ke tong xue/g
```

##### 2.1.3 工厂符号

> `RegExp(pattern [, flags])`

```javascript
let reg3 = RegExp('jing ke tong xue', 'g');
console.log(reg3); // /jing ke tong xue/g
```

#### 2.2 字面量、构造函数、工厂符号在创建正则表达式中的异同

##### 2.2.1 共同点

三种方法都可以创建正则表达式，正则表达式的文本`pattern`为必须参数，标志符`flags`有`g、i、m、y、u`五个可选、可任意搭配参数。

##### 2.2.2 不同点

> 构造函数和工厂符号除了相差一个`new`关键字外没有什么不同，但是不推荐工厂符号的形式创建正则表达式。下面主要说一下字面量和构造函数的形式创建正则表达式的不同之处。

1. 当表达式被赋值时，字面量形式提供正则表达式的`编译（compilation）`状态，当正则表达式保持为常量时使用字面量。例如当你在循环中使用字面量构造一个正则表达式时，正则表达式不会在每一次迭代中都被`重新编译（recompiled）`。

2. 正则表达式对象的构造函数，如`new RegExp('jing ke tong xue')`提供了正则表达式`运行时编译（runtime compilation）`。如果你知道正则表达式模式将会改变，或者你事先不知道什么模式，而是从另一个来源获取，如用户输入，这些情况都可以使用构造函数。

3. 从`ECMAScript 6`开始，当第一个参数为正则表达式而第二个标志参数存在时，`new RegExp(/jing ke tong xue/, 'g')`不再抛出==TypeError== （“当从其他正则表达式进行构造时不支持标志”）的异常，取而代之，将使用这些参数创建一个新的正则表达式。

4. 字面量方式`pattern`中所有字符都是元字符，所以不能进行变量值的拼接。通过构造函数的方式`pattern`中所有字符都是字符串，是可以进行字符串拼接的，同时对于特殊字符也是需要转义的。

* 字符串变量拼接

```javascript
const name = 'jing ke';

// 字符串拼接不会成功
let reg1 = /" + name + " tong xue/g;
console.log(reg1); // /" + name + " tong xue/g

// 字符串拼接可以成功
let reg2 = new RegExp(name + ' tong xue', 'g');
console.log(reg2); // /jing ke tong xue/g
```

* 特殊字符转义

```javascript
const name = 'jing     ke';

// 匹配name，这里jing和ke之间可能有1个或多个空格
let reg1 = /jing\s+ke/g;
console.log(reg1); // /jing\s+ke/g
console.log(reg1.test(name)); // true

// 这里创建的正则表达式和字面量方式创建的结果并不一样
let reg2 = new RegExp('jing\s+ke', 'g');
console.log(reg2); // /jings+ke/g
console.log(reg2.test(name)); // false

// 这里我把reg3稍稍改造了一下，结果就和reg1一样了
let reg3 = new RegExp('jing\\s+ke', 'g');
console.log(reg3); // /jing\s+ke/g
console.log(reg3.test(name)); // true
```

> 再看一个特殊字符转义的例子

```javascript
// 写一个正则表达式匹配反斜杠 \
const str = '\\'; // 这里str就是 \，反斜杠有特殊意义，下文介绍基本元字符会讲
// 字面量方式
let reg1 = /\\/g;
console.log(reg1); // /\\/g
console.log(reg1.test(str)); // true

// 为什么是4个反斜杠，详见下文元字符介绍。自己在控制台试试1个，2个，3个会报什么错
let reg2 = new RegExp('\\\\', 'g');
console.log(reg2); // /\\/g
console.log(reg2.test(str)); // true
```

### 三. 正则表达式中特殊字符

#### 3.1 标志字符
标志字符 | 含义
---|---
g | 全局匹配，找到所有匹配，而不是在第一个匹配后停止。
i | 忽略大小写。
m | 多行，将开始和结束字符（^和$）视为在多行上工作（也就是，分别匹配每一行的开始和结束（由 \n 或 \r 分割），而不只是只匹配整个输入字符串的最开始和最末尾处。
u | ES6新增，Unicode，将模式视为Unicode序列点的序列。
y | ES6新增，粘性匹配，仅匹配目标字符串中此正则表达式的lastIndex属性指示的索引(并且不尝试从任何后续的索引匹配)。

#### 3.2 基本元字符

基本元字符 | 含义
---|---
\. | 匹配除了换行符之外的任何单个字符
\\ | 在非特殊字符之前的反斜杠表示下一个字符是特殊的，不能从字面上解释。例如，没有前`\`的`b`通常匹配小写`b`，无论它们出现在哪里。如果加了`\`,这个字符变成了一个特殊意义的字符，反斜杠也可以将其后的特殊字符，转义为字面量。例如，模式 `/a*/` 代表会匹配0个或者多个a。相反，模式 `/a\*/` 将`*`的特殊性移除，从而可以匹配像 `a*` 这样的字符串。
\| | 逻辑或操作符。
[...] | 定义一个字符集合，匹配字符集合中的一个字符，在字符集合里面像 `.`，`\`这些字符都表示其本身。
[^...] | 对上面一个集合取非。
\- | 定义一个区间，例如[A-Z]，其首尾字符在 ASCII 字符集里面。

#### 3.3 数量元字符

量词 | 含义
---|---
* | 等价于`{0,}`，表示出现任意次，可以不出现。
+ | 等价于`{1,}`，表示至少出现一次，上不封顶。
? | 等价于`{0, 1}`表示出现一次或零次。
{m} | 等价于`{m, m}`，标志正好出现`m`次，不能多也不能少。
{m,} | 表示至少出现 `m` 次，上不封顶。

> 量词后面加`?`可以实现惰性匹配，对应关系如下：

贪婪量词 | 惰性量词
--- | ---
* | *?
+ | +?
? | ??
{m} | {m}?
{m,} | {m,}?

#### 3.4 锚字符（位置元字符）

字符 | 含义
---|---
^ | 单独使用匹配表达式的开始。匹配字符串的开头，在多行检索中，匹配一行的开头。
$ | 匹配表达式的结束。匹配字符串的结尾，在多行检索中，匹配一行的结尾。
\b | 匹配一个单词的边界，简而言之，就是位于字符\w和字符`\W`之间的位置，或位于字符`\w`和字符串的开头或结尾之间的位置（但需要注意的是在字符组内`[\b]`匹配的是退格符）。
\B | 匹配非单词边界。
(?=p) | 匹配 `p` 前面的位置。零宽正向先行断言，要求接下来的字符都与`p`匹配，但不能包括匹配p的那些字符。
(?!p) | 匹配不是 `p` 前面的位置。零宽负向先行断言，要求接下来的字符不与`p`匹配。

#### 3.5 特殊元字符

字符 | 含义
---|---
\d | 等价于`[0-9]`，表示一位数字s。
\D | 等价于`[^0-9]`，表示一位非数字。除了ASCⅡ数字之外的任何字符。
\s | 等价于`[\t\v\n\r\f]`，表示空白符，包括空格，水平制表符`（\t）`，垂直制表符`（\v）`，换行符`（\n`），回车符`（\r）`，换页符`（\f）`。
\S | 等价于`[^\t\v\n\r\f]`，表示非空白符。
\w | 等价于`[0-9a-zA-Z]`，表示数字大小写字母和下划线。
\W | 等价于`[^0-9a-zA-Z]`，表示非单词字符。

### 四. 正则表达式的一些属性

#### 4.1 RegExp.lastIndex

* `lastIndex` 是正则表达式的一个可读可写的整型属性，用来指定下一次匹配的起始索引。

* 只有正则表达式使用了表示全局检索的 "g" 标志时，该属性才会起作用。

> 如果 lastIndex 大于字符串的长度，则 `regexp.test` 和 `regexp.exec` 将会匹配失败，然后 lastIndex 被设置为 0。

> 如果 lastIndex 等于字符串的长度，且该正则表达式匹配空字符串，则该正则表达式匹配从 lastIndex 开始的字符串。

> 如果 lastIndex 等于字符串的长度，且该正则表达式不匹配空字符串 ，则该正则表达式不匹配字符串，lastIndex 被设置为 0。

> 否则，lastIndex 被设置为紧随最近一次成功匹配的下一个位置。

请看如下示例代码：

```javascript
const str = 'jing ke tong xue jing ke tong xue jing ke tong xue jing ke tong xue';
let reg = /jing ke tong xue/g;
// lastIndex从0开始
console.log(reg.lastIndex); // 0
console.log(reg.test(str)); // true
console.log(reg.lastIndex); // 16
console.log(reg.test(str)); // true
console.log(reg.lastIndex); // 33
// lastIndex可修改
reg.lastIndex = 0;
console.log(reg.lastIndex); // 0
console.log(reg.test(str)); // true
console.log(reg.lastIndex); // 16
console.log(reg.test(str)); // true
console.log(reg.lastIndex); // 33
console.log(reg.test(str)); // false
console.log(reg.lastIndex); // 50
console.log(reg.test(str));// true
console.log(reg.lastIndex); // 67
console.log(reg.test(str));// false
// 上一次匹配失败，lastIndex重置为0
console.log(reg.lastIndex); // 0
console.log(reg.test(str));// true
```

#### 4.2 RegExp.prototype.global

* `global`属性表明正则表达式是否使用了 "g" 标志。

* `global`的值是布尔对象，如果使用了 "g" 标志，则返回`true`；否则返回`false`。 "g" 标志意味着正则表达式应该测试字符串中所有可能的匹配。

* `global`是一个正则表达式实例的只读属性。`RegExp.prototype.global`的`writable`、`enumerable`、`configurable`属性均为`false`，无法直接更改此属性。

请看如下示例代码：

```javascript
let reg1 = /jing ke tong xue/;
console.log(reg1.global); // false.

let reg2 = /jing ke tong xue/g;
console.log(reg2.global); // true
```

#### 4.3 RegExp.prototype.ignoreCase

* `ignoreCase`属性表明正则表达式是否使用了 "i" 标志。

* `ignoreCase`的值是布尔对象，如果使用了"i" 标志，则返回`true`；否则，返回`false`。"i"标志意味着在字符串进行匹配时，应该忽略大小写。
* `ignoreCase`是正则表达式实例的只读属性。`RegExp.prototype.ignoreCase`的`writable`、`enumerable`、`configurable`属性均为`false`，无法直接更改此属性。

请看如下示例代码：

```javascript
let reg1 = /jing ke tong xue/;
console.log(reg1.ignoreCase); // false.

let reg2 = /jing ke tong xue/i;
console.log(reg2.ignoreCase); // true
```

#### 4.4 RegExp.prototype.multiline

* `multiline`属性表明正则表达式是否使用了 "m" 标志。

* `multiline`的值是布尔对象，如果使用了"m" 标志，则返回`true`；否则，返回`false`。"m" 标志意味着一个多行输入字符串被看作多行。

* `multiline`是正则表达式实例的一个只读属性。`RegExp.prototype.multiline`的`writable`、`enumerable`、`configurable`属性均为`false`，无法直接更改此属性。


请看如下示例代码：

```javascript
let reg1 = /jing ke tong xue/;
console.log(reg1.multiline); // false.

let reg2 = /jing ke tong xue/m;
console.log(reg2.multiline); // true
```

#### 4.5 RegExp.prototype.unicode

* `unicode`属性表明正则表达式带有"u" 标志。

* `unicode`的值是Boolean，并且如果使用了 "u" 标志则为`true`；否则为`false`。"u" 标志开启了多种Unicode相关的特性。使用 "u" 标志，任何Unicode 代码点的转义都会被解释。

* `unicode`是正则表达式独立实例的只读属性。`RegExp.prototype.unicode`的`writable`、`enumerable`属性为`false`，`configurable`属性均为`true`，无法直接更改此属性。

请看如下示例代码：

```javascript
let reg1 = /jing ke tong xue/;
console.log(reg1.unicode); // false.

let reg2 = /jing ke tong xue/u;
console.log(reg2.unicode); // true
```

再看如下示例代码：

```javascript
// 定义一个四个字节的 UTF-16 编码的字符
const str = '\uD83D\uDC2A';

let reg1 = /^\uD83D/;
// ES5不支持四个字节的 UTF-16 编码，会将其识别为两个字符，故而输出 true
console.log(reg1.test(str)); // true

let reg2 = /^\uD83D/u;
// 加了u修饰符以后，ES6 就会识别其为一个字符，故而输出false
console.log(reg2.test(str)); // false
```

#### 4.6 RegExp.prototype.sticky

* `sticky`属性表明正则表达式带有"y" 标志。

* `sticky`的值是 `Boolean` ，并且如果使用了"y"标志则为`true`；否则为`false`。"y" 标志指示搜索是否具有粘性，仅从正则表达式的lastIndex 属性表示的索引处为目标字符串匹配（并且不会尝试从后续索引匹配）。

* `sticky`是正则表达式独立实例的只读属性。`RegExp.prototype.sticky`的`writable`、`enumerable`属性为`false`，`configurable`属性均为`true`，无法直接更改此属性。


请看如下示例代码：

```javascript
let reg1 = /jing ke tong xue/;
console.log(reg1.sticky); // false.

let reg2 = /jing ke tong xue/y;
console.log(reg2.sticky); // true
```

再看如下示例代码：

```javascript
const str = 'test jing ke tong xue test jing ke tong xue';

let reg1 = /jing ke tong xue/;
// 没有y标识符，lastIndex始终为0，但是reg1并不具有粘性，并不从lastIndex号为开始
console.log(reg1.lastIndex); // 0
console.log(reg1.test(str)); // true
reg1.lastIndex = 5;
console.log(reg1.lastIndex); // 5
console.log(reg1.test(str)); // true
// 由于没有y标识符，在正则表达式匹配之后，lastIndex又被重置为0
console.log(reg1.lastIndex); // 0
console.log(reg1.test(str)); // true

let reg2 = /jing ke tong xue/y;
// 第一次匹配将从0号位开始
console.log(reg2.lastIndex); // 0
console.log(reg2.test(str)); // false
// 第二次一次匹配将从5号位开始
reg2.lastIndex = 5;
console.log(reg2.lastIndex); // 5
console.log(reg2.test(str)); // true
// 下一次匹配将从21号位开始
console.log(reg2.lastIndex); // 21
console.log(reg2.test(str)); // false
// 上一次匹配失败，lastIndex重置为0；下一次匹配将从0号位开始
console.log(reg2.lastIndex); 
```

#### 4.7 RegExp.prototype.source

* `source` 属性返回一个值为当前正则表达式对象的模式文本的字符串。该字符串不会包含正则字面量两边的斜杠以及任何的标志字符。

请看如下示例代码：

```javascript
let reg = /jing ke tong xue/gimuy;
console.log(reg.source); // jing ke tong xue
// source无法被修改，不会生效
reg.source = 'tong xue jing ke';
console.log(reg); // /jing ke tong xue/gimuy
```

#### 4.8 RegExp.prototype.flags

* `flags`为`ES6`新增属性，返回一个由当前正则表达式对象的标志组成的字符串。

* 标志以字母表顺序排序。（从左到右`gimuy`）。
 
* `flags`是正则表达式独立实例的只读属性。`RegExp.prototype.flags`的`writable`、`enumerable`属性为`false`，`configurable`属性均为`true`，无法直接更改此属性。


请看如下示例代码：

```javascript
let reg = /jing ke tong xue/gimuy;
console.log(reg.flags); // jing ke tong xue
// flags无法被修改，不会生效
reg.flags = 'g';
console.log(reg); // /jing ke tong xue/gimuy
```

flags属性为ES6新增属性，ES6以前可用如下Polyfill。

```javascript
if (RegExp.prototype.flags === undefined) {
  Object.defineProperty(RegExp.prototype, 'flags', {
    writable: false, // 默认为false，写上便于理解
    enumerable: false, // 默认为false，写上便于理解
    configurable: true,
    get: function() {
      return this.toString().match(/[gimuy]*$/)[0];
    }
  });
}
```

### 五. 正则表达式的一些方法

#### 5.1 RegExp.prototype.test

* 语法 `reg.test(str)`。`str`：用来与正则表达式匹配的字符串。

* `test()` 方法执行一个检索，用来查看正则表达式与指定的字符串是否匹配。

* 如果正则表达式与指定的字符串匹配 ，返回`true`，否则`false`。

```javascript
const str = 'jing ke tong xue';
let reg1 = /^jing/;
// 判断str是不是以jing开头
console.log(reg1.test(str)); // true

let reg2 = /^ke/;
// 判断str是不是以ke开头
console.log(reg2.test(str)); // fase
```

在设置有全局标志`g`的正则使用test方法。

> 如果正则表达式设置了全局标志，test方法的执行会改变正则表达式的 lastIndex 属性。连续的执行test方法，后续的执行将会从 lastIndex 处开始匹配字符串。如果 test 方法返回匹配失败 false，lastIndex 属性将会被重置为0。

```javascript
const str = 'jing ke tong xue jing ke tong xue jing ke tong xue';
let reg = /jing ke/g;

console.log(reg.lastIndex); // 0
console.log(reg.test(str)); // true
console.log(reg.lastIndex); // 7
console.log(reg.test(str)); // true
console.log(reg.lastIndex); // 24
console.log(reg.test(str)); // true
console.log(reg.lastIndex); // 41
console.log(reg.test(str)); // false
console.log(reg.lastIndex); // 0
```

#### 5.2 RegExp.prototype.exec

* 语法`reg.exec(str)`。`str`：要匹配正则表达式的字符串。 `exec()` 方法在一个指定字符串中执行一个搜索匹配。返回一个结果数组或 `null`。

* 如果匹配成功，`exec()` 方法返回一个数组，并更新正则表达式对象的属性。返回的数组将完全匹配成功的文本作为第一项，将正则括号里匹配成功的作为数组填充到后面。

* 如果匹配失败，`exec()` 方法返回 `null`。

> 💥如果你只是需要第一个匹配结果，你可能想要使用 RegExp.exec() 。

> 💥如果你想要获得捕获组，并且设置了全局标志，你需要用 String.prototype.match() 。

```javascript
const str = 'jing ke tong xue jing ke tong xue jing ke tong xue';
let reg = /jing ke/g;

console.log(reg.lastIndex); // 0
console.log(reg.exec(str)); // ["jing ke", index: 0, input: "jing ke tong xue jing ke tong xue jing ke tong xue", groups: undefined]
console.log(reg.lastIndex); // 7
console.log(reg.exec(str)); // ["jing ke", index: 17, input: "jing ke tong xue jing ke tong xue jing ke tong xue", groups: undefined]
console.log(reg.lastIndex); // 24
console.log(reg.exec(str)); // ["jing ke", index: 34, input: "jing ke tong xue jing ke tong xue jing ke tong xue", groups: undefined]
console.log(reg.lastIndex); // 41
console.log(reg.exec(str)); // null
console.log(reg.lastIndex); // 0
```

从上述代码看出，在使用exec时，经常需要配合使用while循环使用：

```javascript
const str = 'jing ke tong xue jing ke tong xue jing ke tong xue';
let reg = /jing ke/g;

let result;
// 当reg.exec(str)返回null时结束循环，与上面代码结果一致
while (result = reg.exec(str)) {
    console.log(`result: ${result}`, `\nindex: ${result.index}`, `\nlastIndex: ${reg.lastIndex}`, `\ninput: ${result.input}`);
}

// result: jing ke 
// index: 0 
// lastIndex: 7 
// input: jing ke tong xue jing ke tong xue jing ke tong xue
// index.html:132 result: jing ke 

// index: 17 
// lastIndex: 24 
// input: jing ke tong xue jing ke tong xue jing ke tong xue
// index.html:132 result: jing ke 

// index: 34 
// lastIndex: 41 
// input: jing ke tong xue jing ke tong xue jing ke tong xue
```

#### 5.2 RegExp.prototype.toString

* 语法`reg.toString()`

* `toString()` 返回一个表示该正则表达式的字符串。

* `RegExp` 对象覆盖了 `Object` 对象的 `toString()` 方法，并没有继承 `Object.prototype.toString()`。对于 `RegExp` 对象，`toString()` 方法返回一个该正则表达式的字符串形式，同时会将flags属性按照字母表顺序重排。

```javascript
let reg = /jing ke tong xue/yguim;
console.log(reg.toString()); // /jing ke tong xue/gimuy
```

### 六. 4种适用于模式匹配的String方法

> 这里介绍的4种String方法只是和模式匹配相关的内容，与模式匹配不相关的内容下面并不会提及，[如果想了解更多请移步这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)。


#### 6.1 String.prototype.search

* 语法`str.search(regexp)`。

> `regexp`：一个正则表达式（`regular expression`）对象。如果传入一个非正则表达式对象，则会使用 `new RegExp(obj)` 隐式地将其转换为正则表达式对象。

* `search()` 方法执行正则表达式和 String对象之间的一个搜索匹配。当你想要知道字符串中是否存在某个模式（`pattern`）时可使用 `search`，类似于正则表达式的 `test` 方法。

* 如果匹配成功，则 search() 返回正则表达式在字符串中首次匹配项的索引。否则，返回 -1。

> 💥如果你需要知道一个字符串是否匹配一个正则表达式 `RegExp` ，可使用 `search()` 。

```javascript
const str = 'jing ke tong xue jing ke tong xue jing ke tong xue jing ke tong xue';
let reg = /jing ke tong xue/g;
console.log(str.search(reg)); // 0

// 由于条件判断的时候喜欢用true或false, 改造如下, 详情请自行学习JavaScript位(~)运算符
console.log(!!~str.search(reg)); // true
```

#### 6.2 String.prototype.match

* 语法`str.match(regexp)`。

> `regexp`：一个正则表达式对象。如果传入一个非正则表达式对象，则会隐式地使用 `new RegExp(obj)` 将其转换为一个 `RegExp` 。如果你未提供任何参数，直接使用 `match()` ，那么你会得到一个包含空字符串的 `Array ：[""]`。

* 当一个字符串与一个正则表达式匹配时， `match()` 方法检索匹配项。如果正则表达式不包含 `g` 标志，则 `str.match()` 会返回和 `RegExp.exec()` 相同的结果。而且返回的 `Array` 拥有一个额外的 `input` 属性，该属性包含被解析的原始字符串。另外，还拥有一个 index 属性，该属性表示匹配结果在原字符串中的索引（以0开始）。

* 如果正则表达式包含 `g` 标志，则该方法返回一个 `Array` ，它包含所有匹配的子字符串而不是匹配对象。捕获组不会被返回(即不返回`index`属性和`input`属性)。如果没有匹配到，则返回 `null` 。

在下例中，使用 `match` 查找 "March" 紧跟着 1 个或多个数值字符，再紧跟着一个逗号“,”，再紧跟着一个或多个空格，接着跟着4个数值字符。正则表达式包含 i 标志，因此大小写会被忽略。第二个包含 g 标志会进行全局匹配。

```javascript
// 啊，在2018年3月31日的下午，诗性大发，赋诗两句😎
// 🙈一支穿云箭, 千军万马来相见🙈
const str = 'In March 31, 2018, An arrow through the clouds to meet thousands upon thousands of horses and soldiers, In March 31, 2018';
let reg1 = /In (march \d+, (\d{4}))/i;
let match1 = str.match(reg1);
console.log(match1);
// 以下是logs内容
// => [
// =>   "In March 31, 2018",
// =>   "March 31, 2018",
// =>   "2018",
// =>   index: 0,
// =>   input: "In March 31, 2018, An arrow through the clouds to meet thousands upon thousands of horses and soldiers.",
// =>   groups: undefined
// => ]

// 📒释疑📌
// "In March 31, 2018" 是整个匹配
// "March 31, 2018" 是被(March \d+, (\d*))捕获的内容
// "2018" 是被(\d{4}) 捕获的内容
// index: 0 匹配开始的索引
// input: ... 是被解析的原始字符串 

// 这个和上面的差别就是flags多了个g标识符
let reg2 = /In (march \d+, (\d{4}))/ig;
let match2 = str.match(reg2);
// 多了一个标识符,返回的结果是一个包含匹配字符的数组
console.log(match2); // ["In March 31, 2018", "In March 31, 2018"]
```

号外，`match` 方法除了支持正则表达式对象作为参数外，还支持非正则表达式对象作为参数。

```javascript
// 这个例子就不给答案了，好记性不如烂笔头，有心的同学请在控制台输出结果查看区别😄
const str1 = "NaN means not a number. Infinity contains -Infinity and +Infinity in JavaScript.";
const str2 = "4 is 4, 10 is 10, 14 is 14, 40 is 40.";
const str3 = "The contract was declared null and void.";
console.log(str1.match("number"));
console.log(str1.match("NaN"));
console.log(str1.match(NaN));
console.log(str1.match("Infinity"));
console.log(str1.match(Infinity));
console.log(str1.match(+Infinity));
console.log(str1.match(-Infinity));
console.log(str2.match("10"));
console.log(str2.match(10));
console.log(str2.match("-10"));
console.log(str2.match(-10));
console.log(str2.match("+10"));
console.log(str2.match(+10));
console.log(str3.match("null"));
console.log(str3.match(null));
```

#### 6.3 String.prototype.split

* 语法`str.split([separator[, limit]])`。

> `separator`: 指定表示每个拆分应发生的点的字符串。separator 可以是一个字符串或正则表达式。 如果纯文本分隔符包含多个字符，则必须找到整个字符串来表示分割点。如果在str中省略或不出现分隔符，则返回的数组包含一个由整个字符串组成的元素。如果分隔符为空字符串，则将str原字符串中每个字符的数组形式返回。

> `limit`: 一个整数，限定返回的分割片段数量。当提供此参数时，split 方法会在指定分隔符的每次出现时分割该字符串，但在限制条目已放入数组时停止。如果在达到指定限制之前达到字符串的末尾，它可能仍然包含少于限制的条目。新数组中不返回剩下的文本。

`split` 方法接受两个参数，返回一个数组。第一个是用来分割字符串的字符或者正则，如果是空字符串则会将元字符串中的每个字符以数组形式返回，第二个参数可选作为限制分割多少个字符，也是返回的数组的长度限制。有一个地方需要注意，用捕获括号的时候会将匹配结果也包含在返回的数组中。

```javascript
const str = 'jing ke tong xue jing ke tong xue jing ke tong xue';
let reg1 = /\s/;
console.log(str.split(reg1));
// => ["jing", "ke", "tong", "xue", "jing", "ke", "tong", "xue", "jing", "ke", "tong", "xue"]
console.log(str.split(" "));
// => ["jing", "ke", "tong", "xue", "jing", "ke", "tong", "xue", "jing", "ke", "tong", "xue"]

let reg2 = /\s+(tong)\s+/;
console.log(str.split(reg2));
// => ["jing ke", "tong", "xue jing ke", "tong", "xue jing ke", "tong", "xue"]

let reg3 = /\s+tong\s+/;
console.log(str.split(reg3));
// => ["jing ke", "xue jing ke", "xue jing ke", "xue"]

console.log(str.split(' tong '));
// => ["jing ke", "xue jing ke", "xue jing ke", "xue"]
```

#### 6.4 String.prototype.replace

***该方法并不改变调用它的字符串本身，而只是返回一个新的替换后的字符串。***

***在进行全局的搜索替换时，正则表达式需包含 g 标志。***

* 语法 `str.replace(regexp|substr, newSubStr|function)`。

> `regexp (pattern)`: 一个RegExp 对象或者其字面量。该正则所匹配的内容会被第二个参数的返回值替换掉。

> `substr (pattern)`: 一个要被 newSubStr 替换的字符串。其被视为一整个字符串，而不是一个正则表达式。仅仅是第一个匹配会被替换。

> `newSubStr (replacement)`: 用于替换掉第一个参数在原字符串中的匹配部分的字符串。该字符串中可以内插一些特殊的变量名。参考下面的使用字符串作为参数。

> `function (replacement)`: 一个用来创建新子字符串的函数，该函数的返回值将替换掉第一个参数匹配到的结果。参考下面的指定一个函数作为参数。

`replace`方法接受两个参数，第一个是要被替换的文本，可以是正则也可以是字符串，如果是字符串的时候不会被转换成正则，而是作为检索的直接量文本。第二个是替换成的文本，可以是字符串或者函数，字符串可以使用一些特殊的变量来替代前面捕获到的子串。返回一个部分或全部匹配由替代模式所取代的新的字符串。

如果使用字符串作为参数时替换字符串可以插入下面的特殊变量名：

变量名 | 代表的值
---|---
$$ | 插入一个 "$"。
$& | 插入匹配的子串。
$` | 插入当前匹配的子串左边的内容。
$' | 插入当前匹配的子串右边的内容。
$n | 假如第一个参数是 RegExp对象，并且 n 是个小于100的非负整数，那么插入第 n 个括号匹配的字符串。

如果是函数的话，当匹配执行后， 该函数就会执行。 函数的返回值作为替换字符串。另外要注意的是， 如果第一个参数是正则表达式， 并且其为全局匹配模式， 那么这个方法将被多次调用， 每次匹配都会被调用。


变量名 | 代表的值
---|---
match | 匹配的子串。（对应于上述的$&。）
p1, p2, ... | 假如replace()方法的第一个参数是一个RegExp 对象，则代表第n个括号匹配的字符串。（对应于上述的$1，$2等。）
offset | 匹配到的子字符串在原字符串中的偏移量。（比如，如果原字符串是“abcd”，匹配到的子字符串是“bc”，那么这个参数将是1）
string | 被匹配的原字符串。

```javascript
const str = '2018-03-31';
let reg = /^(\d{4})\D(\d{2})\D(\d{2})$/;
function replacer (match, p1, p2, p3, offset, string) {
    console.log([match, p1, p2, p3, offset, string]);
    return [p1, p2, p3].join('/')
}
console.log(str.replace(reg, replacer));

// => ["2018-03-31", "2018", "03", "31", 0, "2018-03-31"]
// => 2018/03/31
```

由于内容比较多，文章写的太长也不便于阅读，所以本篇至此结束。后面估计还有两篇完结，到时候补了再更。

- ***第一篇完结(就是本文): JavaScript正则表达式学习笔记(一) - 理论基础***
-  ***[第二篇待更：JavaScript正则表达式学习笔记(二) - 打怪升级](https://juejin.im/post/5ac971bb6fb9a028bf059372)***
-  ***第三篇待更：JavaScript正则表达式学习笔记(三) - 终章***