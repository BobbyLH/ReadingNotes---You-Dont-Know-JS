# 类型 - JS中，变量(variables)没有类型，只有值(values)有类型

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




