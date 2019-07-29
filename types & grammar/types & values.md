# 类型和值(Types & Values) - JS中，变量(variables)没有类型，只有值(values)有类型

## 基本类型(Scalar Primitives)
- null
- undefined
- boolean
- number
- string
- object
- symbol

### undefined
- undefined区别undeclared，前者是一个变量已经被声明(declared)在作用域中，后者表示没有被声明的变量
  ```javascript
  var a;
  typeof a; // undefined
  typeof b; // undefined -- 安全边界

  // 块级作用域中使用let const 声明的变量若提前使用typeof获取类型, 安全边界消失
  { typeof c; let c = 123 }

  typeof window === 'undefined'; // 作为客户端的判断依然有效
  ```

### number
- JS中没有区分integer和float(fraction)

- JS中的number 是基于JEEE 754标准，通常叫float-point，而JS中的number使用的就是其double precision(64-bit binary)的标准格式
  * 基于这个标准，才会出现 `0.1 + 0.2 === 0.3; //false` 的浮点数精度丢失的情况
  * 具体原因是因为0.1这个浮点数在取整验算中，有些对应的小数是无法写全的，因为JEEE 754 64bit只允许存在52位fraction，超过的部分进一舍零
- `0.42` 和 `.42` 都是有效的数字, `42.0` 和 `42.` 也是一样

- *很大* 或 *很小*的数字会被转换成 `指数形式(exponent form)`, 也可以用**toExponential**内建方法来转换成指数形式的字符串
  ```javascript
  var a = 5E10;
  a; // 50000000000
  a.toExponential(); // '5e+10'
  var b = a * a;
  b; // 2.5e+21
  var c = 1 / b; // 4e-22
  ```

- **toFixed** 和 **toPrecision** 很相似，前者是指定小数点个数，后者则是指定有多少有效数字

- 虽然不必通过变量保存某个number后再用 `.` 运算符获取这些方法，但要小心 `.` 运算符一开始会被当成是number字面量的一部分，如果可能的话，才会被解释成属性获取
  ```javascript
  (42).toFixed(3);  // 42.000
  0.42.toFixed(3);  // 0.420
  42..toFixed(3);  // 42.000
  42.toFixed(3); // SyntaxError
  42 .toFixed(3); // 42.000  留意空格
  ```

- **Number.EPSILON**
  ```javascript
  // 用这个预定义的公差值，我们能够在JS中比较放心的比较浮点数是否相等
  if (!Number.EPSILON) Number.EPSILON = Math.pow(2, -52);
  ```

- JS中最大的数约为 **1.798e+308**，即**Number.MAX_VALUE**

- 最小的数不是负数，是 **Number.MIN_VALUE** (5e-324)

- ES6定义了**Number.MAX_SAFE_INTERGER** 为 2^53 - 1

- **Number.MIN_SAFE_INTERGER** 为 -(2^53 - 1)

- 上面这些数字的定义主要是为了考虑JS有可能会处理数据库64位的ID

- **Infinity** 定义了无限大

- **Number.isInteger** 和 **Number.isSafeInteger** 可以测试number值是否是整型和安全整型

- JS 按位运算符的操作范围在 `2^31 - 1` 到 `-2^31` 之间(32bit)，因此也能将number的范围进一步缩小
  ```javascript
  var a = Math.pow(2, 33);
  var b = a | 9829;
  b; //9829
  ```
- NaN
  ```javascript
  NaN === NaN; // false
  Object.is(NaN, NaN); // true
  function mockIs (a, b) {
    if(a === 0 && b === 0) return  1/a === 1/b;

    if (a !== a) return b !== b;

    return a === b;
  };
  ```
- -0
  ```javascript
  -0 === 0; // true
  Object.is(-0, 0); // false
  ```

### string
- String在某些方面和Array有共同点，特别当Array是一组仅包含字符(character)的数组时

  - 都有length属性
  - 都有indexOf、concat等内建方法
    ```javascript
    var a = 'foo';
    var b = ['f', 'o', 'o'];

    a.indexOf('o'); // 1
    b.indexOf('o'); // 1
    a.length; // 3
    b.length; // 3
    ```
- 但实际上String 和 String[]有很大的区别
  - 字符串是不可变的(immutable)
    ```javascript
    var a = 'foo';
    var b = ['f', 'o', 'o'];

    a[1] = 'O' // 'foo'
    b[1] = 'O' // ['f', 'O', 'o']
    ```
  - 老旧的IE不支持`a[1]`的形式，用a.chartAt(1)代替
  - 并且大部分的Array内建方法都会改变原数组，比如splice、pop、push，但所有的String内建方法都只是返回一个新String，并不修改目标
- 当然, 我们能够[借用](https://codepen.io/bobby_li/pen/wLMVEq?editors=1111)一些不会修改原目标的Array的内建方法应用到String上

----

## 复杂类型(Compound Value)
1. Array
2. Object
3. Function
4. Error
5. Date
6. RegExp

### Array
- **delete** - 该方法会删除slot值，但是依然保留slot，因此会创建出松散(sparse)的数组
  ```javascript
  var a = [1,2,3];
  delete a[2];

  a.length; // 3
  a[2]; // undefined
  ```
- **array-like** 类数组
  ```javascript
  function () { arguments };
  document.querySelectorAll('div');
  ```
- 类数组转换为数组
  - Array.prototype.slice.call()
  - Array.from()

----

## 类型的识别
1. typeof
  - 只能满足基本类型的检测
  - 对于null的检查是一个bug
    ```javascript
    typeof null === 'object'; // true
    ```
  - 能满足对于对于function
    ```javascript
    typeof function () {} === 'function'; // true
    ```
2. instanceof
3. Object.prototype.toString

----

## 值和引用(Value & Reference)
- JS中, 决定值拷贝还是地址引用的是值的类型

- 基本类型 都是值拷贝(value-copy)

- object类型和function类型 都是地址引用(reference-copy), 但实际上指针指向的也是底层的值(underlying value)，而不是其他的变量或引用

- 设定`a`是数组，因此传递的是地址，在函数内对参数进行push操作，会影响到引用地址的值
  ```javascript
  function foo (a) {
    a.push(3);
    a.length = 0;
    a.push(1,2,3)
  };

  var a = [9,8];
  foo(a);
  a; // [1,2,3] 
  ```

- 即使是原始类型的包装对象，也无法形成地址引用，改变包装对象的值
  ```javascript
  function foo (a) {
    a = a + 1;
    console.log(a); // 4
  };

  var a = new Number(3);
  foo(a);
  a; // 3
  ```