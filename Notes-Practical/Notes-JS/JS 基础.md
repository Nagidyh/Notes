## 基础

### 数组

#### IndexOf

搜索指定元素位置，返回索引值。

```javascript
var arr = [10, 20, '30', 'xyz'];
arr.indexOf(10); // 元素10的索引为0
arr.indexOf(20); // 元素20的索引为1
arr.indexOf(30); // 元素30没有找到，返回-1
arr.indexOf('30'); // 元素'30'的索引为2
```

#### Slice

截取数组元素，返回包含截取到的元素的新数组。

```javascript
var arr = ['A', 'B', 'C', 'D', 'E', 'F', 'G'];
arr.slice(0, 3); // 从索引0开始，到索引3结束，但不包括索引3: ['A', 'B', 'C']
arr.slice(3); // 从索引3开始到结束: ['D', 'E', 'F', 'G']
```

#### pop/push/shift/unshift

老朋友，不谈了。

- push 尾部追加
- pop 尾部弹出
- shift 头部删除
- unshift 头部添加

#### Sort

排序，默认按升序排序。

#### Reverse

反转数组。

#### Splice

从指定的索引开始删除若干元素，然后再从该位置添加若干元素：

```javascript
var arr = ['Microsoft', 'Apple', 'Yahoo', 'AOL', 'Excite', 'Oracle'];
// 从索引2开始删除3个元素,然后再添加两个元素:
arr.splice(2, 3, 'Google', 'Facebook'); // 返回删除的元素 ['Yahoo', 'AOL', 'Excite']
arr; // ['Microsoft', 'Apple', 'Google', 'Facebook', 'Oracle']
// 只删除,不添加:
arr.splice(2, 2); // ['Google', 'Facebook']
arr; // ['Microsoft', 'Apple', 'Oracle']
// 只添加,不删除:
arr.splice(2, 0, 'Google', 'Facebook'); // 返回[],因为没有删除任何元素
arr; // ['Microsoft', 'Apple', 'Google', 'Facebook', 'Oracle']
```

#### Concat

连接两个数组，返回新数组。

```javascript
var arr = ['A', 'B', 'C'];
var added = arr.concat([1, 2, 3]);
added; // ['A', 'B', 'C', 1, 2, 3]
arr; // ['A', 'B', 'C']
```

#### Join

类似 PHP 的 implode 方法，将数组转为字符串用分隔符连接，返回字符串。

```javascript
var arr = ['A', 'B', 'C', 1, 2, 3];
arr.join('-'); // 'A-B-C-1-2-3'
```

### 对象

简单记录一下 JS 对象的几个特殊性质。

定义 JS 对象：

```javascript
var xiaoming = {
    name: '小明',
    birth: 1990,
    school: 'No.1 Middle School',
    height: 1.70,
    weight: 65,
    score: null,
    'middle-school': 'MD'
};
```

一般声明的属性，可以通过 `.` 操作符来访问，但是当对象中的属性名存在特殊字符时，在内部声明的时候就需用使用 `''` 来包裹属性名。这时使用 `.` 字符就无法访问到属性了。

```javascript
xiaoming.name; // 小明
// xiaoming.middle-school 无法通过语法检查器
xiaoming['middle-school']; // MD
xiaoming['name']; // 小明
```

另外，在 JS 中访问一个不存在的属性， JS 并不会报错，而会返回 undefined ；

### Map/Set

其实在 JS 中，使用 `{}` 包裹的对象就可以视为是一个 MAP ，但是它有一个问题，就是他的 Key 值必须为 String ，然而实际上使用 Number 也是一种常见的情况。

因此，在 ES6 规范中，引入了 Map 和 Set 两个数据结构的对象实现。

如果浏览器不支持 ES6 规范，会报 ReferenceError 错误。

都引入 Map 和 Set 了，自然也一同引入了 iterable 类型来遍历这两个数据结构。

具有 iterable 类型的集合可以使用 `for (var item of items)` 来遍历。

也可以直接使用 iterable 自带的 `ForEach` 方法来使用，它接受一个回调函数，在每次迭代中会回调这个函数，并传入参数。

`iterable.forEach( function(element, index, array) { ... })`

`map.forEach( function(value, key, map)){ ... }`

`set.forEach( function(element, sameElement, set)){ ... }`

## 函数

### 定义与调用

#### 定义与调用

```javascript
// e.g. 
var energy = function (m, c) { 
    return m * c**2;
}
// 等价于
function enegry (m, c) {
    return m * c**2;
}
// 使用函数名称调用，未传入的参数会被定义为 undefined，进行运算的结果为 NaN 。
enegry(2,4); // 32
```

#### 形参声明

```javascript
// ES6 before 
// 可以使用 arguments 获取调用时传入的所有参数
function func (a,b) {
    return arguments.lenght; // 注意： arguments 本质并不是一个 Array 。
}
func(1,2,3,4); // 4
// ES6 after
function func2 (a, ...rest) {
    return rest;
}
func(1,2,3,4); // [2,3,4]
```

### 变量作用域与解构赋值

使用 `var` 声明的变量都是**函数级作用域级**的变量，即在声明变量的函数体外不可使用，在函数中嵌套声明同名变量，会由**内而外**的查找变量的声明。

```javascript
function func() {
    var a = 1;
    function inserFunc() {
        var b = a + 1;
    }
    var c = b ; // RefernceError !
}
```

#### 变量提升

JavaScript的函数定义有个特点，它会先扫描整个函数体的语句，把所有申明的变量“提升”到函数顶部。

```javascript
function func() {
    var x = 'Hi, ' + y; // 不会报错，JS引擎会将 y 的声明提升到顶部。
    var y = 'Bob';
}
// 要注意的是， JS 引擎会提升变量的声明，但不会提升变量的复制
// 即 x 的值还是 Hi, undefined 
```

由于JavaScript的这一怪异的“特性”，我们在函数内部定义变量时，请严格遵守“**在函数内部首先申明所有变量**”这一规则。

最常见的做法是用一个`var`申明函数内部用到的所有变量

```javascript
function foo() {
    var
        x = 1, // x初始化为1
        y = x + 1, // y初始化为2
        z, i; // z和i为undefined
    // 其他语句:
    for (i=0; i<100; i++) {
        ...
    }
}
```

#### 命名空间

在 Web 开发中，不在任何函数内定义的变量就具有全局作用域。即在全局对象 `window` 下。

JavaScript 实际上**只有一个全局作用域**。任何变量（函数也视为变量），如果没有在当前函数作用域中找到，就会继续往上查找，最后如果在全局作用域中也没有找到，则报 `ReferenceError` 错误。

全局变量会绑定到 `window` 上，不同的JavaScript文件如果使用了相同的全局变量，或者定义了相同名字的顶层函数，都会造成命名冲突，并且很难被发现。

减少冲突的一个方法是把自己的所有变量和函数全部绑定到一个全局变量中。例如：

```javascript
// 唯一的全局变量MYAPP:
var MYAPP = {};

// 其他变量:
MYAPP.name = 'myapp';
MYAPP.version = 1.0;

// 其他函数:
MYAPP.foo = function () {
    return 'foo';
};
```

把自己的代码全部放入唯一的名字空间 `MYAPP` 中，会大大减少全局变量冲突的可能。

#### let & const 

由于JavaScript的变量作用域实际上是函数内部，我们在`for`循环等语句块中是无法定义具有局部作用域的变量的：

```javascript
'use strict';

function foo() {
    for (var i=0; i<100; i++) {
        //
    }
    i += 100; // 仍然可以引用变量i
}
```

为了解决块级作用域，ES6引入了新的关键字 `let` ，用 `let` 替代 `var` 可以申明一个块级作用域的变量：

```javascript
'use strict';

function foo() {
    var sum = 0;
    for (let i=0; i<100; i++) {
        sum += i;
    }
    // SyntaxError:
    i += 1;
}
```

由于 `var` 和 `let` 声明的是变量，如果要声明一个常量，在 ES6 之前是不行的，我们通常用全部大写的变量来表示“这是一个常量，不要修改它的值”：

```javascript
var PI = 3.14;
```

ES6标准引入了新的关键字 `const` 来定义常量，`const` 与 `let` 都具有块级作用域：

```javascript
'use strict';

const PI = 3.14;
PI = 3; // 某些浏览器不报错，但是无效果！
PI; // 3.14
```

#### 解构赋值

从 ES6 开始，JavaScript 引入了解构赋值，可以同时对一组变量进行赋值。

分别例举几个常见的用法

```javascript
// 从数组抽取元素
let [x, [y, z]] = ['hello', ['JavaScript', 'ES6']];
let [, , z] = ['hello', 'JavaScript', 'ES6']; // 忽略前两个元素，只对z赋值第三个元素

// 从对象抽取属性
var person = {
    name: '小明',
    age: 20,
    gender: 'male',
    passport: 'G-12345678',
    school: 'No.4 middle school',
    address: {
        city: 'Beijing',
        street: 'No.1 Road',
        zipcode: '100001'
    }
};
var {name, married, passport:id, address: {city, zip}} = person;
name; // 小明
married; // undefined
id; // G-12345678
address; // Uncaught ReferenceError: address is not defined
city; // Beijing
zip; // undefined
```

有些时候，如果变量已经被声明了，再次赋值的时候，正确的写法也会报语法错误：

```javascript
// 声明变量:
var x, y;
// 解构赋值:
{x, y} = { name: '小明', x: 100, y: 200};
// 语法错误: Uncaught SyntaxError: Unexpected token =
```

这是因为JavaScript引擎把 `{` 开头的语句当作了块处理，于是 `=` 不再合法。解决方法是用小括号括起来：

```javascript
({x, y} = { name: '小明', x: 100, y: 200});
```

- 交换两个变量的值

  ```javascript
  var x = 1, y = 2;
  [x, y] = [y, x];
  ```
  
- 获取当前页面的域名和路径

  ```javascript
  var {hostname:domain, pathname:path} = location;
  ```

- 接受对象作为参数时解构对象

  ```javascript
  function BuildDate({year, month, day, hour=0, minute=0, second=0}) {
      return new Date(year + '-' + month + '-' + day + ' ' + hour + ':' + minute + ':' + second);
  }
  ```

题外话，关于交换两个变量的奇淫巧技：

```javascript
b=[a,a=b][0]
b=a+b-(b=a)
```

### 对象方法

在 javascript 中，对象的属性也可以绑定为一个函数，即为对象的方法。

#### this

javascript 的 this 指向一个历史遗留问题，简单来讲，javascript 的 **this 指向取决于调用函数的对象**，分为以下几个情况

```javascript
function getAge() {
    var y = new Date().getFullYear();
    return y - this.birth;
}
var xm = {
    name: '小明',
    birth: 1990,
    age: getAge
}
var fn = xm.age;
```

- strict 模式下
  - `getAge()` this 指向 undefined，所以返回值为 NaN 。
  - `xm.age()` this 指向 xm，返回值正常为 2022 - 1999 。
  - `fn()` this 指向 undefined，所以返回值为 NaN 。
- 非 strict 模式下
  - `getAge()` this 会指向 `window` 对象，结果仍是 NaN 。
  - `xm.age()` this 正常指向 xm 。
  - `fn()` this 会指向 `window` 对象，结果仍是 NaN 。

在实际的使用过程中，会有更为复杂的情况导致无法直观的看到函数的调用对象，比如：

```javascript
var xm = {
    name: '小明',
    birth: 1990,
    age: funciton() {
		function getAgeFromBirth() {
            var y = new Date().getFullYear();
            return y - this.birth;
        }
		return getAgeFromBirth(); // 此处调用方法时，this 又指向了 undefined 或 window
	}
}
// 所以一般需要使用其他方法处理 this 时，需要先将需要处理的 this 捕获，防止被错误的替换。
var xm = {
    name: '小明',
    birth: 1990,
    age: funciton() {
    	var that = this; // 声明 that 捕获真实调用对象。
		function getAgeFromBirth() {
            var y = new Date().getFullYear();
            return y - that.birth;
        }
		return getAgeFromBirth(); // 此处调用方法时，this 又指向了 undefined 或 window
	}
}
// 用 var that = this;，你就可以放心地在方法内部定义其他函数，而不是把所有语句都堆到一个方法中。
```

#### apply & call 

javascript 也提供了控制 this 指向的方法，使用方法如下，每一个函数都拥有这两个方法。

它有两个参数，第一个指 this 指向的对象，第二个为函数的参数列表。

```javascript
function getAge() {
    var y = new Date().getFullYear();
    return y - this.birth;
}

var xiaoming = {
    name: '小明',
    birth: 1990,
    age: getAge
};

xiaoming.age(); // 25
getAge.apply(xiaoming, []); // 25, this指向xiaoming, 参数为空
```

另一个与 `apply()` 类似的方法是 `call()`，唯一区别是：

- `apply() `把参数打包成 `Array` 再传入；
- `call() `把参数按顺序传入。

使用 apply 和利用 JS 语言本身的灵活性，可以很简单的就实现装饰器设计，甚至为内置函数提供装饰，而不需要使用代理模式。

```javascript
var oldAlert = window.alert;
window.alert = function() {
    console.log('前方高能');
    return oldAlert.apply(null, arguments);
}
```

### 高阶函数

#### map / reduce 

map 和 reduce 方法都是 Array 的方法，可以更简洁的处理数组数据。

- `var arr = [x1,x2,x3].map( x => x**2 )` 使用方法处理数组中每个元素并返回处理后的数组。
- `var sum = [x1,x2,x3].reduce( x,y => x + y )` reduce 会为方法提供两个参数，分别是上次处理的结果和数组值，返回遍历整个数组后最后一个结果。

> 参考资料： [MapReduce: Simplified Data Processing on Large Clusters](http://research.google.com/archive/mapreduce.html) 

#### filter

用于过滤 Array 中的元素，并返回剩下的元素。

和 `map()` 类似，`Array` 的 `filter()` 也接收一个函数。和 `map()` 不同的是，`filter()` 把传入的函数依次作用于每个元素，然后根据返回值是 `true `还是 `false` **决定保留还是丢弃该元素**。

```javascript
// E.g. 删除偶数保留奇数
[1,2,3,4,5,6,7,8,9].filter( x => x%2!=0 );
```

`filter` 方法为回调方法提供三个参数

- element 当前元素
- index 当前元素索引
- self 数组本身

```javascript
// 使用 filter 去重
[1,2,3,2,3,4,2,5,6,5].filter( element,index,self => self.indexOf(element) === index );
```

### 闭包

闭包是函数语言的特性，函数作为一种基本类型 [Function] 可以作为参数传递，也可以作为返回值返回。

简而言之，所谓闭包就是一个携带状态的函数，比如利用闭包实现一个求和函数，但不需要马上获得结果。

```javascript
function lazy_sum(arr) {
    return () => arr.reduce( (x,y) => x + y );
}
let 
	arr = [1,2,3,4,5,6],
    arrSum = lazy_sum(arr);
arrSum();	// 21
arr[0] = 10;
arrSum();	// 30
```

利用函数传递和 lambda 表达式，能够实现如此简洁优雅的代码，就是函数式编程的魅力。

闭包特性也可以用于模拟 OOP 的封装机制，主要的目的是保证在实现业务的过程中尽量保证每个业务的独立性，不要为了封装而封装。

比如实现一个计数器

```javascript
function create_counter(initial) {
    let x = initial || 0;
    return {
        inc: function () {
            x += 1;
            return x;
        }
    }
}
let c1 = create_counter();
c1.inc(); // 1
c1.inc(); // 2
c1.inc(); // 3

let c2 = create_counter(20);
c2.inc(); // 21
```

这就实现了对 x 的封装，外部根本无法访问在 create_counter 方法中的局部变量，只能通过 inc 方法对 x 进行递增。

### generator

