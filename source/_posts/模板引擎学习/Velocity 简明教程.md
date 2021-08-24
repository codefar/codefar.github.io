# Velocity 简明教程

# 简介
Velocity在Apache官网地址：http://velocity.apache.org/
Velocity是一个基于Java的模板引擎。它允许任何人使用简单而强大的模板语言来引用Java代码中定义的对象。

作为一个模块引擎，Velocity可以用作品前后端分离的MVC展现层，也可以用来生成SQL、 PostScript、XML等。


# 程序基本用法


```
    Velocity.init();

        VelocityContext context = new VelocityContext();
        context.put("name", "Velocity");

        Template template = null;
        try {
            template = Velocity.getTemplate("mytemplate.vm");
        } catch( ResourceNotFoundException e ) {
            // couldn't find the template
        } catch( ParseErrorException pee ) {
            // syntax error: problem parsing the template
        } catch( MethodInvocationException mie ) {
            // something invoked in the template
            // threw an exception
        } catch( Exception e ) {

        }

        StringWriter sw = new StringWriter();
        template.merge(context, sw);
        
        
        
   //
   
   Properties props = new Properties();
        props.load(this.getClass().getResourceAsStream("/vm.properties"));
        VelocityEngine ve = new VelocityEngine(props);

        ve.init();

  }
```


# 模板基本语法
## 变量
使用$符声明变量，可以声明变量也可以对变量进行赋值(变量是弱类型的)。还可以使用$取出在VelocityContext容器中存放的值。

```
#set($name =“velocity”)
#set($foo.name = $bar.name)
#set($foo.name = $bar.getName($arg))
#set($foo = 123)
#set($foo = [“foo”,$bar])
```

变量的使用  
在模板文件中使用$name 或者 ${name} 来使用定义的变量。推荐使用 ${name} 这种格式，

## 循环
在Velocity中可以使用循环语法遍历集合，语法结构如下：

```
#foreach($item in $list)
 $item
 $velocityCount 
#end
```
Velocity会创建一个 $velocityCount 的变量作为计数，从 1 开始，每次循环都会加 1。

## 条件控制语法
在Velocity中可以使用条件语法对流程进行控制


```
#if(condition)
...dosonmething...
#elseif(condition)
...dosomething...
#else
...dosomething...
#end
```

## 关系操作符

Velocity 引擎提供了 AND、OR 和 NOT 操作符，分别对应&&、||和! 例如：

```
#if($foo && $bar)
#end
```
## 宏

Velocity 中的宏可以理解为函数定义。定义的语法如下：

```
#macro(macroName arg1 arg2 …)
...
#end
```

调用这个宏的语法是：

```
#macroName(arg1 arg2 …)
```

这里的**参数之间使用空格隔开**，下面是定义和使用 Velocity 宏的例子：


```
#macro(sayHello $name)
hello $name
#end
#sayHello(“velocity”)
```
输出的结果为 hello velocity

## #parse 和 #include

#parse 和 #include 指令的功能都是在外部引用文件，而两者的区别是，#parse 会将引用的内容当成类似于**源码文件**，会将内容在引入的地方进行解析，#include 是将引入文件当成资源文件，会将引入内容原封不动地以**文本输出**。分别看以下例子：

foo.vm 文件：
```
#set($name =“velocity”)
```
include.vm：
```
#include(“foo.vm”)
```
输出结果为：#set($name =“velocity”)

parse.vm：
```
#parse(“foo.vm”)
```
输出结果为：velocity  

## 在web项目中使用Velocity
velocity只是一个模板引擎，在web项目中使用Velocity还得添加一个HTTP框架来处理请求和转发，apache提供了velocity-tools，其提供了VelocityViewServlet，也可继承VelocityViewServlet,从而实现自己的HTTP框架
一般都是继承VelocityViewServlet，重写handleRequest方法，在其中存入公共的参数.

通过继承或直接使用VelocityViewServlet，可以在管理的vm文件中获得request、session与application对象，也可以直接获取在这几个域对象中保存的值，获取的顺序与EL表达式获取的顺序类似:

```
${request} --> ${session} --> ${application}

比如${testArr}，获取testArr属性，velocity会在velocity的context中寻找。没找到在request域中找，没找到在session中找.
```


下面将通过实例的方式讲解如何在web项目中使用Velocity
首先引入velocity-tools及其依赖的相关jar包，然后分为如下4步：

### 继承VelocityViewServlet
通过继承VelocityViewServlet重写handleRequest方法，可以自定义转发规则

```
public class MyVelocityViewServlet extends VelocityViewServlet {
    @Override
    protected Template handleRequest(HttpServletRequest request, HttpServletResponse response, Context ctx) {
        // 往Context容器存放变量
        ctx.put("fullName","lixiaolin");
        // 也可以往request域中存值
        request.setAttribute("anotherName","xlli");
        // forward到指定模板
        return getTemplate("test.vm");
    }
}
```
### 配置web.xml
对自定义的VelocityViewServlet配置就像配置普通的Servlet一样，如下:

```
<servlet>
    <servlet-name>MyVelocityServlet</servlet-name>
    <servlet-class>com.lxl.velocity.MyVelocityViewServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>MyVelocityServlet</servlet-name>
    <url-pattern>/servlet/myVelocityServlet</url-pattern>
</servlet-mapping>
```

参考
-  http://velocity.apache.org/engine/2.0/user-guide.html
-  https://www.ibm.com/developerworks/cn/java/j-lo-velocity1/
-  
