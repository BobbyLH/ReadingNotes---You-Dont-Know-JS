# 语法(Grammer)

## 陈述句(Statements) 和 表达式(Expression)
- Statements 类似句子，是一个由单词组成表达想法的formation, 会包含一个或多个的词组phrase，有的词组能独立成形，有的需要依赖其他单词/词组才有意义，这就是所谓的grammer

- Expression 类似词组，Operator 类似连词/标点符号

---

## 陈述句完成值(Statement Completion Values)
- `console.log`的完成值是undefined
  ```javascript
  console.log(3); // 3 undefined
  ```

- 下例中`if(...)`的完成值是5，即最后一行表达式的值，可以用`eval(...)`来验证
  ```javascript
  var a, b;
  if (true) b = 3 + 2; // 5

  a = eval('if (true) b = 3 + 2'); // 5
  a; // 5
  ```

---

## 表达式副作用(Expression Side Effect)
- `++` - 会显示返回操作对象的值
  - 下面代码示例，按照操作符运算优先级，赋值运算符的优先级远低于后置++，为什么b的值依然是12不是13呢？但d的值就是15而不是14呢？—— **因为`后置++`运算符先返回没有运算前的结果，然后才进行自增的动作, 而`(a++, a)`返回的则是后置++运算符计算完后`a`的值**
  ```javascript
  var a = 10;
  a++;
  ++a;
  var b = a++;
  b; // 12
  var c = ++a;
  c; // 14
  var d = (a++, a);
  d; // 15
  ```

  - 下面的`++e++` 会报错，因为 e++ 先执行，然后返回21，++21是错误的表达式，因为原始类型不能改变自身，且++运算符的目标只能是变量
  ```javascript
  var e = 20;
  ++e++; // ReferenceError
  ```
  ![avatar](./assets/coercion_grammer_err_++.png)

- `delete` - 会显示返回操作的结果 true/false
  ```javascript
  var obj = { a:1 } ;
  delete obj; // false
  delete obj.a; // true
  ```

- `=` - 会显示返回赋值的结果
  ```javascript
  var a, b, c;
  a = b = c = 33; //33
  ```

  ```javascript
  function test (str) {
    let match;
    // (match = str.match(/[test]/ig)) 部分的返回值充当了 && 运算符的第二个操作数
    if (str && (match = str.match(/[test]/ig)))a {
      return match; 
    }  
  }
  ```

- `-- += -=` 等运算符也是具有副作用的表达式，并且也有相应的返回值

----

## 运算符

### 运算符优先级(Operator Precedence) - Which one bind first before others
- `()` 优先级最高

- `,` 优先级比 `=` 低

- `&&` 优先级高于 `||`
  ```javascript
  false && true || true;   // true
  true || false && false;   // true
  ````

### 运算符结合律(Associativity / Grouping) - How multiple operator expressions are implicitly grouped
- 区别执行顺序(JS 依然是从左往右执行 left-to-right processing): [Example](https://codepen.io/bobby_li/pen/bPVaNR?editors=1111)

- 左结合(left-associative)
  - `&&` 和 `||` 是左结合模式 —— 虽然无论左还是右对其值都不影响
  ```javascript
  var a = true;
  var b = false;
  var c = true;

  a && b && c; // false
  (a && b) && c; // false
  a && (b && c); // false

  a || b || c; // true
  (a || b) || c; // true
  a || (b || c); // true
  ```

- 右结合(right-associative)
  - ` ? : ` 三元运算符(ternary)是右结合模式
  ```javascript
  var a = true;
  var b = 'b';
  var c = false;
  var d = 'd';
  var e = 'e';

  a ? b : c ? d : e; // 'b'
  (a ? b : c) ? d : e; // 'd'
  a ? b : (c ? d : e);  // 'b'
  ```

  - `=` 赋值运算符 右结合
  ```javascript
  var a, b, c;
  a = b = c = 42;

  // 上面的代码就相当于下面的加了()的代码
  a = (b = (c = 42));
  ```

- 当左右结合在一块时
  ```javascript
  var a = 42;
  var b = 'foo';
  var c = false; 

  var d = a && b || c ? c || b ? a : c && b : a;
  d; //42

  // 上面的代码就相当于下面的加了()的代码
  ((a && b) || c) ? ((c || b) ? a : (c && b)) : a
  ```