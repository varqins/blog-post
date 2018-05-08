
![JavaScript正则表达式学习笔记(二) - 打怪升级.png](https://note.youdao.com/yws/public/resource/f7844a944d640f82e89c257a5e696c4a/xmlnote/021AD5F2F1D942F8968D212917302205/3098)

> 本文接上篇，基础部分相对薄弱的同学请移步[《JavaScript正则表达式学习笔记(一) - 理论基础》](https://juejin.im/post/5ac184a9f265da237b223cb0)。上文介绍了**8**种JavaScript正则表达式的属性，本文还会追加介绍几种JavaScript正则表达式的属性(注意是非标准属性，但很好用)。

### 一. 上文回顾

> 本文会用到上篇文章部分内容，所以简单回顾下。

#### 1.1 JavaScript正则表达式标志符

1. `g`: 全局匹配，即找到所有匹配的。对应属性`RegExp#global`。

2. `i`: 忽略字母大小写。对应属性`RegExp#ingoreCase`。

3. `m`: 多行匹配，只影响^和$，二者变成行的概念，即行开头和行结尾。对应属性`RegExp#multiline`。

4. `u`: ES6新增。含义为“Unicode 模式”，用来正确处理大于\uFFFF的 Unicode 字符。也就是说，会正确处理四个字节的 UTF-16 编码。对应属性`RegExp#unicode`。

5. `y`: ES6新增。y修饰符的作用与g修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，g修饰符只要剩余位置中存在匹配就可，而y修饰符确保匹配必须从剩余的第一个位置开始，这也就是“粘连”的涵义。对应属性`RegExp#sticky`。

#### 1.2 适用于Javascript正则表达式的方法

上篇文章[《JavaScript正则表达式学习笔记(一) - 理论基础》](https://juejin.im/post/5ac184a9f265da237b223cb0)介绍了适用于JavaScript正则表达式模式匹配的相关API共有**6**种，`RexExp`提供2个，`String`提供4个，如下：

```javascript
1. RegExp#test    // 适用于：验证、提取
2. RegExp#exec    // 适用于：验证、提取
3. String#search  // 适用于：验证、提取
4. String#match   // 适用于：验证、提取
5. String#split   // 适用于：切分
6. String#replace // 适用于：提取、替换
```

### 二. JavaScript正则表达式的四种操作

正则表达式是用于匹配字符串中字符组合的模式, 其核心内容便是模式匹配。也就是说，不论进行那种操作，首先要有模式匹配，有了模式匹配之后，才能进行验证、替换、切分、提取这四种操作。

#### 2.1 验证

验证应该是前端程序员写正则表达式用的最多的方法吧，比如表单验证之类的。可以实现验证的方法有4种。

光说不练假把式，光练不说傻把式，又练又说才是真把式💪。那么，请看下面的例子🌰：

##### 2.1.1 使用`test`方法

如果正则表达式与指定的字符串匹配 ，返回true，否则false。得到的结果可以直接使用。

```javascript
const str = 'jing ke tong xue 666';

let reg1 = /\d{4}/;
let res1 = reg1.test(str);
console.log(res1);
// => false

let reg2 = /ke/;
let res2 = reg2.test(str);
console.log(res2);
// => true

let reg3 = /\d{3}/;
let res3 = reg3.test(str);
console.log(res3);
// => true
```

下面我们看一个匹配身份证的示例：

```javascript
const str1 = '411199909096896';
const str2 = '411425199909096896';
const str3 = '41142519990909689x';
const str4 = '41142519990909689X';
const reg = /^(\d{15}|\d{17}[\dxX])$/;
let res1 = reg.test(str1);
let res2 = reg.test(str2);
let res3 = reg.test(str3);
let res4 = reg.test(str4);
console.log(res1, res2, res3, res4);
// => true true true true
```


不过好在网上有很多可视化工具供我们使用，看一下下图可能就豁然开朗。

![可视化视图](https://note.youdao.com/yws/public/resource/f7844a944d640f82e89c257a5e696c4a/xmlnote/AE364651074149339E67B2DFBC1A0CEF/2591)

看图分析：这里竖杠`|`的优先级最低，所以正则分成了两部分`\d{15}`和`\d{17}[\dxX]`

* `\d{15}` 表示15位连续数字

* `\d{17}[\dxX]` 表示17位连续数字，最后一位可以是数字，也可以大写或小写字母 "x"

下面我们再看一个稍微复杂一点的案例，匹配IPV4地址：

```javascript
const str = '192.168.0.1';
let reg = /^((0{0,2}\d|0?\d{2}|1\d{2}|2[0-4]\d|25[0-5])\.){3}(0{0,2}\d|0?\d{2}|1\d{2}|2[0-4]\d|25[0-5])$/;
let res = reg.test(str);
console.log(res);
// => true
```

上面那个正则表达式初看起来有点吓人哈🙈，这就是传说中的写正则不难，读正则难。先看下面可视化视图：

![可视化视图](https://note.youdao.com/yws/public/resource/f7844a944d640f82e89c257a5e696c4a/xmlnote/5CD53535CE3A4C4385BC3EC26D04D1B2/2534)

此正则表达式主体结构大致如下：

`((…)\.){3}(…)`

两个(...)是相同的内容，因此文字描述为`3位数字.3数字.3位数字.3位数字`。

下面我们具体分析`(...)`也就是`(0{0,2}\d|0?\d{2}|1\d{2}|2[0-4]\d|25[0-5])`，单独看一下这段代码的可视化视图：

![可视化视图](https://note.youdao.com/yws/public/resource/f7844a944d640f82e89c257a5e696c4a/xmlnote/81DFF4C644BD4F008E72BCBE4AA0CD26/2564)

看图分析，它是一个多选结构，分成5个部分：

* `0{0,2}\d`: 匹配一位数，包括 "0" 补齐的。比如，"9"、"09"、"009"

* `0?\d{2}`: 匹配两位数，包括 "0" 补齐的，也包括一位数

* `1\d{2}`: 匹配 "100" 到 "199"

* `2[0-4]\d`: 匹配 "200" 到 "249"

* `25[0-5]`: 匹配 "250" 到 "255"

不要被看起来复杂的正则表达式吓到，乍看起来复杂的不要不要的，那我们就把它像洋葱一样，一层一层剥开它的心，拥抱它，然后扒光它。

##### 2.1.2 使用`exec`方法

如果匹配成功，返回一个`Array`，否则返回null。得到的结果两次取反取得true或者false使用。

```javascript
const str = 'jing ke tong xue 666';

let reg1 = /\d{4}/;
let res1 = reg1.exec(str);
console.log(res1);
console.log(!!res1);
// => null
// => false

let reg2 = /ke/;
let res2 = reg2.exec(str);
console.log(res2);
console.log(!!res2);
// => ["ke", index: 5, input: "jing ke tong xue 666", groups: undefined]
// => true

let reg3 = /\d{3}/;
let res3 = reg3.exec(str);
console.log(res3);
console.log(!!res3);
// => ["666", index: 17, input: "jing ke tong xue 666", groups: undefined]
// => true
```

##### 2.1.3 使用`search`方法

如果匹配成功，返回正则表达式在字符串中首次匹配项的索引(大于等于0)，否则，返回 -1。如果为了和上面的方法保持一致返回true或false，这里需要借助一次按位非操作符(~)。

```javascript
const str = 'jing ke tong xue 666';

let reg1 = /\d{4}/;
let res1 = str.search(reg1);
console.log(res1);
console.log(!!~res1);
// => -1
// => flase

let reg2 = /ke/;
let res2 = str.search(reg2);
console.log(res2);
console.log(!!~res2);
// => 5
// => true

const reg3 = /\d{3}/;
let res3 = str.search(reg3);
console.log(res3);
console.log(!!~res3);
// => 17
// => true
```

##### 2.1.4 使用`match`方法

如果匹配成功，返回一个`Array`，否则返回null。得到的结果两次取反取得true或者false使用。

```javascript
const str = 'jing ke tong xue 666';

let reg1 = /\d{4}/;
let res1 = str.match(reg1);
console.log(res1);
console.log(!!res1);
// => null
// => flase

let reg2 = /ke/;
let res2 = str.match(reg2);
console.log(res2);
console.log(!!res2);
// => ["ke", index: 5, input: "jing ke tong xue 666", groups: undefined]
// => true

let reg3 = /\d{3}/;
let res3 = str.match(reg3);
console.log(res3);
console.log(!!res3);
// => ["666", index: 17, input: "jing ke tong xue 666", groups: undefined]
// => true
```

#### 2.2 替换

*有时候找到了对象往往不是我们最终的目的，找到了对象做点什么才是我们想要的😍。*

这里我们看一看找到后的替换操作，我本人的常用场景就是格式化日期和删除空格。

```javascript
// 把YYYY/MM/DD格式的日期替换成YYYY-MM-DD格式。
const str1 = 'jing-ke-tong-xue';

let res1 = str1.replace(/-/g, ' ');
console.log(res1);
// => jing ke tong xue

// 删除前后空格, 为了直观这里把前后空格替换成“删除了的空格”
const str2 = ' jing ke tong xue ';
let res2 = str2.replace(/^\s|\s$/g, '删除了的空格');
console.log(res2);
// => 删除了的空格jing ke tong xue删除了的空格

// 据说下面这种方法速度比较快
const str3 = ' jing ke tong xue ';
let res3 = str3.replace(/^\s/, '删除了的空格').replace(/\s$/, '删除了的空格');
console.log(res3);
// => 删除了的空格jing ke tong xue删除了的空格
```

#### 2.3 切分

切萝卜切萝卜切切切，包饺子包饺子捏捏捏🐷。在这里，所谓切分就是把字符串切的一段一段的。

```javascript
// 按空格切分
const str = 'jing ke tong xue';

console.log(str.split(/\s/));
// => ["jing", "ke", "tong", "xue"]

const str2 = 'jing * ke ￥ tong ^xue';
console.log(str2.split(/\s\*\s|\s￥\s|\s\^/))
// => ["jing", "ke", "tong", "xue"]
```
#### 2.4 提取

有时候验证、替换、切分都不是目的，我们需要提取出来对我们有用的信息，那么就需要提取了。此时需要用到下篇会写到的“括号”（分组引用或者分组捕获）。

这里有一个正则将会贯穿本小节，我们先提前分析一下：

```javascript
// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg = /(\d{4})\D(\d{2})\D(\d{2})/g;
```

可视化视图：

![可视化视图](https://note.youdao.com/yws/public/resource/f7844a944d640f82e89c257a5e696c4a/xmlnote/682FA291FDBF4FA2B86096DAFCFC7EDC/2665)

看图说话：从上图可知这段正则的意思就是：`4个数字`后跟一个`非数字`再跟`2个数字`再跟一个`非数字`再跟`2个数字`。形如：`4个数字-2个数字-2个数字`格式。

2.4.1 使用`exec`方法

```javascript
const str = "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串";
// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg = /(\d{4})\D(\d{2})\D(\d{2})/g;
// 上篇文章有说：`exex`方法匹配到一次就会返回结果，想要下一个结果必须再次调用
console.log(reg.exec(str));
console.log(reg.exec(str));
// => ["2018-04-04", "2018", "04", "04", index: 16, input: "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是2018-40-40，提取日期测试字符串", groups: undefined]
// => ["6666-66-66", "6666", "66", "66", index: 34, input: "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串", groups: undefined]
```

2.4.2 使用`match`方法

```javascript
const str = "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串";

// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg1 = /(\d{4})\D(\d{2})\D(\d{2})/;
console.log(str.match(reg1));
// 没有g标识符返回结果与match无异
// => ["2018-04-04", "2018", "04", "04", index: 16, input: "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串", groups: undefined]

// 带有g标识符，结果数组只包含匹配结果
// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg2 = /(\d{4})\D(\d{2})\D(\d{2})/g;
console.log(str.match(reg2));
// => ["2018-04-04", "6666-66-66"]
```

2.4.3 使用`replace`方法

在上篇文章步[《JavaScript正则表达式学习笔记(一) - 理论基础》](https://juejin.im/post/5ac184a9f265da237b223cb0)的理论基础中提到`replace`方法的第二个参数可以是一个函数，函数接收的参数包含我们需要的信息。

```javascript
const str = "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串";

// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg1 = /(\d{4})\D(\d{2})\D(\d{2})/;

let res1 = [];
str.replace(reg1, function(match, year, month, day, offset, string) {
    res1.push(match, year, month, day, offset, string);
});
console.log(res1);
// => ["2018-04-04", "2018", "04", "04", 16, "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串"]

// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg2 = /(\d{4})\D(\d{2})\D(\d{2})/g;

let res2 = [];
str.replace(reg2, function(match, year, month, day, offset, string) {
    res2.push(match);
});
console.log(res2);
// => ["2018-04-04", "6666-66-66"]
```

从输出效果来看，`replace`方法可以达到模拟`match`方法的效果。这就是传说中的***条条大路通罗马***`No roads can't lead to Rome`(哼...哼哼...翻译是我故意的嗷😋)

2.4.4 使用`test`方法

> 此方法会用到`RegExp.$1-$9`这一非标属性，这里只是使用，下文会做介绍。

```javascript
const str = "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串";

// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg = /(\d{4})\D(\d{2})\D(\d{2})/;

reg.test(str);

// RegExp.$1-$9非标属性，但是目的达到了，请勿用于生产环境
let res = [RegExp.$1, RegExp.$2, RegExp.$3];

console.log(res);
// => ["2018", "04", "04"]
```

2.4.4 使用`search`方法

> 此方法会用到`RegExp.$1-$9`这一非标属性，这里只是使用，同样留到下文介绍。

```javascript
const str = "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串";

// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg = /(\d{4})\D(\d{2})\D(\d{2})/;

str.search(reg);

// RegExp.$1-$9非标属性，但是目的达到了，请勿用于生产环境
let res = [RegExp.$1, RegExp.$2, RegExp.$3];

console.log(res);
// => ["2018", "04", "04"]
```

到这里正则表达式的验证、替换、切分、提取已经介绍完了，有些操作是**取巧**的做法，也不建议在生产环境使用，不过某些特殊情况除外。至于哪些情况是特殊情况，具体问题具体分析吧😄。

### 三. 注意要点

在本文 ***1.2*** 小节提到了**6**种可以用于正则操作的方法，`RegExp`提供2种，`String`提供4种。本章节就围绕这几种方法展开。

#### 3.1 `match`方法返回结果的格式不一致问题

> 这个问题上文[《JavaScript正则表达式学习笔记(一) - 理论基础》](https://juejin.im/post/5ac184a9f265da237b223cb0)也有体现，这里再单独拿来说一说，以加深记忆。

```javascript
const str = "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串";

// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg1 = /(\d{4})\D(\d{2})\D(\d{2})/;
console.log(str.match(reg1));
// => ["2018-04-04", "2018", "04", "04", index: 16, input: "提取日期测试字符串，今天的日期是2018-04-04，今天的日期不是6666-66-66，提取日期测试字符串", groups: undefined]

// 这里为了简单测试，只考虑了日期格式，没考虑日期的合理性
let reg2 = /(\d{4})\D(\d{2})\D(\d{2})/g;
console.log(str.match(reg2));
// => ["2018-04-04", "6666-66-66"]

// 这个正则只是来客串说明问题，没有其他意义
let reg3 = /.^/;
console.log(str.match(reg3));
// => null

// 这个正则只是来客串说明问题，没有其他意义
let reg4 = /.^/g;
console.log(str.match(reg4));
// => null
```

* 当没有`g`标识符时，返回的结果为标准匹配格式，包含完整的匹配信息。

* 当有`g`标识符时，返回的结果为所有匹配结果组成的数组。

* 当匹配不成功时，无论有没有标识符（包括`igmyu`的任意组合），都返回`null`。

#### 3.2 `search`和`match`的参数问题

话不多说，先看代码：

```javascript
// 为了说明问题，日期格式选择了用“.”连接，因为“.”在正则中属于元字符
const str = '2018.04.04';

console.log(str.search('.'));
console.log(str.search(/./));
console.log(str.search('\\.'));
console.log(str.search(/\./));
// => 0
// => 0
// => 4
// => 4

console.log(str.match('.'));
console.log(str.match(/./));
console.log(str.match('\\.'));
console.log(str.match(/\./));
// => ["2", index: 0, input: "2018.04.04", groups: undefined]
// => ["2", index: 0, input: "2018.04.04", groups: undefined]
// => [".", index: 4, input: "2018.04.04", groups: undefined]
// => [".", index: 4, input: "2018.04.04", groups: undefined]

console.log(str.replace('.', '-'));
console.log(str.replace( /./, '-'));
console.log(str.replace('\.', '-'));
console.log(str.replace(/\./, '-'));
// => 2018-04.04
// => -018.04.04
// => 2018-04.04
// => 2018-04.04

console.log(str.split('.'));
console.log(str.split(/./));
console.log(str.split('\\.'));
console.log(str.split(/\./));
// => ["2018", "04", "04"]
// => ["", "", "", "", "", "", "", "", "", "", ""]
// => ["2018.04.04"]
// => ["2018", "04", "04"]
```

从上述代码可以看出`search`方法和`match`方法会将接收到的字符串参数转为正则，而`replace`方法和`split`方法不会转换，所以使用时请注意。

#### 3.3 `exec`和`match`的王者之争

**王者之争第一回合：**

```javascript
const str = '2018-04-04';

let reg1 = /\b(\d+)\b/;
let reg2 = /\b(\d+)\b/g;

console.log(reg1.exec(str));
console.log(reg2.exec(str));
// => ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]
// => ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]

console.log(str.match(reg1));
console.log(str.match(reg2));
// => ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]
// ["2018", "04", "04"]
```

* `g`标识符对`exec`方法没有产生影响，但是改变了`match`方法的行为。没主见啊，没主见。

* 没有`g`标识符时，`match`返回标准匹配参数，有`g`标识符时返回了匹配结果的集合。但是对于`exec`方法无论有没有`g`标识符都返回了同样的结果。不智能啊，不智能。

*这一回合实力相差不大，看不出孰优孰劣。*

**王者之争第二回合：**

```javascript
const str = '2018-04-04';

let reg1 = /\b(\d+)\b/;
let reg2 = /\b(\d+)\b/g;

console.log(reg2.lastIndex);
console.log(reg2.exec(str));
// => 0
// => ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]

console.log(reg2.lastIndex);
console.log(reg2.exec(str));
// => 4
// => ["04", "04", index: 5, input: "2018-04-04", groups: undefined]

console.log(reg2.lastIndex);
console.log(reg2.exec(str));
// => 7
// => ["04", "04", index: 8, input: "2018-04-04", groups: undefined]

console.log(reg2.lastIndex);
console.log(reg2.exec(str));
// => 10
// => null

// 上一次匹配失败，这一次从头开始
console.log(reg2.lastIndex);
console.log(reg2.exec(str));
// => 0
// => ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]

// 没有g标识符时match方法每次都从第一位开始匹配
console.log(str.match(reg1));
console.log(str.match(reg1));
// => ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]
// => ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]
```

这一回合没有悬念，`exec`方法胜出。不过上面的写法未免太过繁琐。请看下面当`exec`遇上`while`：

```javascript
const str = '2018-04-04';

let reg = /\b(\d+)\b/g;

// 结合while流程控制语句
let res;
while (res = reg.exec(str)) {
    console.log(reg.lastIndex, res);
}
// => 4 ["2018", "2018", index: 0, input: "2018-04-04", groups: undefined]
// => 7 ["04", "04", index: 5, input: "2018-04-04", groups: undefined]
// => 10 ["04", "04", index: 8, input: "2018-04-04", groups: undefined]
```

王者之争至此结束。`exec`要比`match`强大那么一点哈。

#### 3.4 除了`exec`方法，`g`标识符对`test`方法也有影响

上篇文章曾经提到 **lastIndex  是正则表达式的一个可读可写的整型属性，用来指定下一次匹配的起始索引。** 也就是说，只要正则还是那个正则，当存在`g`标识符的时候`lastIndex`都会做出相应的改变，要匹配的字符串可以不是同一个。

```javascript
const str1 = '2018-04-04';
const str2 = '2018-04-05';
const str3 = '2018-04-06';
const str4 = '2018-04-07';
const str5 = '2018-04-08';

let reg = /\b(\d+)\b/g;

console.log(reg.lastIndex);
console.log(reg.test(str1));
// => 0
// => true

console.log(reg.lastIndex);
console.log(reg.test(str2));
// => 4
// => true

console.log(reg.lastIndex);
console.log(reg.test(str3));
// => 7
// => true

console.log(reg.lastIndex);
console.log(reg.test(str4));
// => 10
// => false

console.log(reg.lastIndex);
console.log(reg.test(str5));
// => 0
// => true
```

当没有`g`操作符时，始终从0开始匹配，这里就不做演示了。

#### 3.5 `split`方法注意事项

```javascript
const str = '2018-04-04';

console.log(str.split('-'));
console.log(str.split('-', 2));
console.log(str.split('-', 10));
// => ["2018", "04", "04"]
// => ["2018", "04"]
// => ["2018", "04", "04"]

let reg1 = /-/;
let reg2 = /(-)/;

console.log(str.split(reg1));
console.log(str.split(reg2));
// => ["2018", "04", "04"]
// => ["2018", "-", "04", "-", "04"]
```

* `split`方法可以接收第二个参数指定返回数组长度，第二个参数只有小于实际返回数组长度时才生效。

* 当`split`接收的正则表达式种包含分组模式时，返回的结果数组包含分组匹配项。

#### 3.6 强大的`replace`方法

`replace`方法不但可以接收一个函数作为第二个参数(前面已经体现，这儿不重复示例)，也可以接收一个字符串作为第二个参数。此处的字符串除了是一个普通的替换字符串之外，也可以是一个特殊变量。本小节以实际示例介绍一下这几个特殊变量（第一篇理论基础有提及）。

变量名 | 代表的值
---|---
$$ | 插入一个 "$"。
$& | 插入匹配的子串。
$` | 插入当前匹配的子串左边的内容。
$' | 插入当前匹配的子串右边的内容。
$n | 匹配第n个分组里捕获的文本，n是不大于100的正整数。

哇哈，是时候表演真正的技术了，下面我们就来看看`replace`的能力。

```javascript
const str1 = '3 6 9';
        
let reg1 = /\d/g;
// 被$包围
console.log(str1.replace(reg1, '$$$&$$'));
// => $3$ $6$ $9$

// 分身
console.log(str1.replace(reg1, '$&$&$&'));
// => 333 666 999

// 分身相加
let reg2 = /(\d)\s(\d)\s(\d)/;
console.log(str1.replace(reg2, '$1$1$1+$2$2$2$2=$3$3$3'));
// => 333+6666=999

// 你爱我我爱你
console.log(str1.replace(reg2, '$1$1$1+$2$2$2=$2$2$2+$1$1$1=$3$3$3=>😍'));
// => 333+666=666+333=999=>😍

const str2 = '3🥕6🌰9';
let reg3 = /🥕|🌰/g;
console.log(str2.replace(reg3, "($&的左边是: $`, 右边是: $')"));
// => 3(🥕的左边是: 3, 右边是: 6🌰9)6(🌰的左边是: 3🥕6, 右边是: 9)9
```

突然间感觉王者之争不应该只让`exec`和`match`参加，`replace`这么强大，也应该参与其中的嘛😄。

#### 3.7 构造函数和字面量问题

这里没有什么悬念，一般建议优先使用字面量的方式创建正则表达式，因为构造函数中需要对元字符转义，会多写很多的反斜杠。当然特殊情况还是要用构造函数。

```javascript
const str = '2018-04-04';

let reg1 = /(\d{4})\D(\d{2})\D(\d{2})/;
console.log(reg1);
console.log(reg1.test(str));
// => /(\d{4})\D(\d{2})\D(\d{2})/
// => true

let reg2 = new RegExp('(\d{4})\D(\d{2})\D(\d{2})');
console.log(reg2);
console.log(reg2.test(str));
// => /(d{4})D(d{2})D(d{2})/
// => false

let reg3 = new RegExp('(\\d{4})\\D(\\d{2})\\D(\\d{2})');
console.log(reg3);
console.log(reg3.test(str));
// => /(\d{4})\D(\d{2})\D(\d{2})/
// => true
```

下面是特殊情况：

```javascript
let name = 'user name';

// user name是一个变量
const str = '2018-04-04 user name';

// 在字面量中，无法实现动态拼接
let reg1 = /(\d{4})\D(\d{2})\D(\d{2})\D + name/;
console.log(reg1);
console.log(reg1.test(str));
// => /(\d{4})\D(\d{2})\D(\d{2})\D + name/
// => flase

let reg2 = new RegExp('(\\d{4})\\D(\\d{2})\\D(\\d{2})\\D' + '(' + name + ')');
console.log(reg2);
console.log(reg2.test(str));
// => /(\d{4})\D(\d{2})\D(\d{2})\D(user name)/
// => true
```

#### 3.8 几个非标属性

上面用到了`RegExp.$1`这一非标属性，所谓非标属性，就是此属性不符合当前任何标准规范。所以，请尽量不要在生产环境中使用，除非特殊情况并且你能保证以后也不会出错。

属性 | 别名 | 说明
---|---|---
RegExp.$1-$9 | 无 | 静态、只读属性。包含括号子串匹配的正则表达式的静态和只读属性。只有在正确匹配的情况下才会改变。虽然括号可以无限，但是此属性最多只能匹配9个。
RegExp.input | RegExp.$_ | 静态属性，含有正则表达式最近一次所匹配的字符串。当正则表达式上搜索的字符串发生该变，并且字符串匹配时，input 属性的值会修改。
RegExp.lastMatch | RegExp['$&'] | 静态、只读属性。含有最近一次匹配到的字符串。属性的值是只读的，会在匹配成功时修改。
RegExp.lastParen | RegExp['$+'] | 静态、只读属性，包含匹配到的最后一个子串。会在匹配成功时修改。
RegExp.leftContext | RegExp['$`'] | 静态、只读属性。含有最新匹配的左侧子串。 会在匹配成功时修改。
RegExp.rightContext | RegExp["$'"] | 静态、只读属性。含有最新匹配的右侧子串。 会在匹配成功时修改。

这几个属性平时也基本用不到，了解了解总是好的，请看下面示例：

```javascript
const str = 'a1b2c3d4e5f6';
let reg = /([a-f])([1-6])/g;

// 为了倒数第二个有输出，这里执行两次exec方法
console.log(reg.exec(str));
console.log(reg.exec(str));
// ["a1", "a", "1", index: 0, input: "a1b2c3d4e5f6", groups: undefined]
// ["b2", "b", "2", index: 2, input: "a1b2c3d4e5f6", groups: undefined]

console.log(RegExp.$1);
console.log(RegExp.$2);
// => b
// => 2

console.log(RegExp.input);
console.log(RegExp.$_);
// => a1b2c3d4e5f6
// => a1b2c3d4e5f6

console.log(RegExp.lastMatch);
console.log(RegExp['$&']);
// => b2
// => b2

console.log(RegExp.lastParen);
console.log(RegExp['$+']);
// => 2
// => 2

console.log(RegExp.leftContext);
console.log(RegExp['$`']);
// => a1
// => a1

console.log(RegExp.rightContext);
console.log(RegExp["$'"]);
// => c3d4e5f6
// => c3d4e5f6
```

### 四. 奇技淫巧

本文写的也挺长的，剩下的准备再写一篇终结JavaScript正则表达式部分的内容。那么本文就先用一个真是案例来做结尾吧。

**JavaScript常用的类型判断实现**

下面这段代码是在某个框架源码中见到的，初见之时倍感惊艳(原谅我入行不久见识短🍷)，其中也用到上文提到的`split`方法，特拿出来分享一下。

```javascript
let utils = Object.create(null);
const types = 'Boolean|Number|String|Function|Array|Date|RegExp|Object|Error';
types.split('|').forEach(type => {
    utils['is' + type] = obj => {
        return Object.prototype.toString.call(obj) == '[object ' + type + ']';
    };
});
console.log(utils.isBoolean('true'));
console.log(utils.isBoolean(true));
```

虽然可以将`Boolean|Number|String|Function|Array|Date|RegExp|Object|Error`存储为数组减少一次`split`切分操作，但是这样似乎多了点黑科技的感觉。

> 由于本同学能力有限，不足之处还望各位大佬同学指正。

***至此，本文完。***

- ***[第一篇完结: JavaScript正则表达式学习笔记(一) - 理论基础](https://juejin.im/post/5ac184a9f265da237b223cb0)***
-  ***第二篇完结(就是本文)：JavaScript正则表达式学习笔记(二) - 打怪升级***
-  ***第三篇待更：JavaScript正则表达式学习笔记(三) - 终章***


