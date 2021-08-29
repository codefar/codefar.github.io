---
layout: post
title: JavaScript学习
date: '2016-06-12 14:01'
---
JavaScript 是世界上最流行的编程语言，HTML 中的脚本必须位于 <script> 与 </script> 标签之间。
脚本可被放置在 HTML 页面的 <body> 和 <head> 部分中。

在 JavaScript 中，用分号来结束语句是可选的
JavaScript 对大小写是敏感的。

### JavaScript 注释
* 单行注释以 // 开头。
* 多行注释以 /* 开始，以 */ 结尾

### JavaScript 变量
在 JavaScript 中我们使用 var 关键词来声明变量：  
var name;  
变量赋值，请使用等号：  
name="Volvo";

* 变量必须以字母开头
* 变量也能以 $ 和 _ 符号开头（不过我们不推荐这么做）
* 变量名称对大小写敏感

### JavaScript 数据类型
字符串、数字、布尔、数组、对象、Null、Undefined
<pre>
var x                // x 为 undefined
var x = 6;           // x 为数字
var x = "Bill";      // x 为字符串
</pre>

字符串可以是引号中的任意文本。您可以使用单引号或双引号：
<pre>
var carname="Bill Gates";
var carname='Bill Gates';
</pre>

JavaScript 数字
JavaScript 只有一种数字类型。数字可以带小数点，也可以不带：
<pre>
var x1=34.00;      //使用小数点来写
var x2=34;         //不使用小数点来写

极大或极小的数字可以通过科学（指数）计数法来书写：
var y=123e5;      // 12300000
var z=123e-5;     // 0.00123
</pre>

JavaScript 布尔 布尔（逻辑）只能有两个值：true 或 false。
<pre>
var x=true
var y=false
</pre>
布尔常用在条件测试中。

JavaScript 数组

<pre>
下面的代码创建名为 cars 的数组：
var cars=new Array();
cars[0]="Audi";
cars[1]="BMW";
cars[2]="Volvo";
或者 (condensed array):
var cars=new Array("Audi","BMW","Volvo");
或者 (literal array):
实例
var cars=["Audi","BMW","Volvo"];
</pre>

JavaScript 对象
对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔：
<pre>
var person={firstname:"Bill", lastname:"Gates", id:5566};
</pre>

Undefined 和 Null
Undefined 这个值表示变量不含有值。
可以通过将变量的值设置为 null 来清空变量。
<pre>
cars=null;
person=null;
</pre>

## JavaScript 对象
JavaScript 中的所有事物都是对象：字符串、数字、数组、日期，等等。
在 JavaScript 中，对象是拥有属性和方法的数据。

属性和方法
属性是与对象相关的值。
方法是能够在对象上执行的动作。

## JavaScript 函数
<pre>
function functionname()
{
这里是要执行的代码
}
//带参数的函数
function myFunction(var1,var2)
{
这里是要执行的代码
}
//带返回值的函数
function myFunction()
{
var x=5;
return x;
}
</pre>


### JavaScript 运算符
### JavaScript 逻辑
