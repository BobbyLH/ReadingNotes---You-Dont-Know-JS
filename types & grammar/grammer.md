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