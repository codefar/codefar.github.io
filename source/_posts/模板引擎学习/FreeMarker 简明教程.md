
[TOC]

# FreeMarker 简明教程

![image](https://www.lifeofpix.com/wp-content/uploads/2018/08/20170423_175910-1600x1200.jpg)

![image](https://www.lifeofpix.com/wp-content/uploads/2018/07/pink-mountains-1600x1067.jpg)
http://freemarker.foofun.cn/index.html


FreeMarker 是一款 模板引擎： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

模板编写为FreeMarker Template Language (FTL)。它是简单的，专用的语言， 不是 像PHP那样成熟的编程语言。 那就意味着要准备数据在真实编程语言中来显示，比如数据库查询和业务运算， 之后模板显示已经准备好的数据。在模板中，你可以专注于如何展现数据， 而在模板之外可以专注于要展示什么数据。

![image](http://freemarker.foofun.cn/figures/overview.png)

这种方式通常被称为 MVC (模型 视图 控制器) 模式，对于动态网页来说，是一种特别流行的模式。 它帮助从开发人员(Java 程序员)中分离出网页设计师(HTML设计师)。设计师无需面对模板中的复杂逻辑， 在没有程序员来修改或重新编译代码时，也可以修改页面的样式。

而FreeMarker最初的设计，是被用来在MVC模式的Web开发框架中生成HTML页面的，它没有被绑定到 Servlet或HTML或任意Web相关的东西上。它也可以用于非Web应用环境中。

# 模板开发指南

## 入门
### 模板 + 数据模型 = 输出

```
<html>
<head>
  <title>Welcome!</title>
</head>
<body>
  <h1>Welcome John Doe!</h1>
  <p>Our latest product:
  <a href="products/greenmouse.html">green mouse</a>!
</body>
</html>
```

### 数据模型一览
### 模板一览
## 数值，类型
### 基本内容
### 类型
## 模板
### 总体结构
### 指令
#### 基本指令
### 表达式
### 插值
## 其它
### 自定义指令
### 在模板中定义变量
### 命名空间
### 空白处理
### 替换(方括号)语法

# 程序开发指南

## 入门

- 创建 Configuration 实例
- 创建数据模型
- 获取模板
- 合并模板和数据模型
- 将代码放在一起

### 创建 Configuration 实例
首先，你应该创建一个 freemarker.template.Configuration 实例， 然后调整它的设置。Configuration 实例是存储 FreeMarker 应用级设置的核心部分。同时，它也处理创建和 缓存 预解析模板(比如 Template 对象)的工作。

也许你只在应用(可能是servlet)生命周期的开始执行一次：


```
// Create your Configuration instance, and specify if up to what FreeMarker
// version (here 2.3.22) do you want to apply the fixes that are not 100%
// backward-compatible. See the Configuration JavaDoc for details.
Configuration cfg = new Configuration(Configuration.VERSION_2_3_22);

// Specify the source where the template files come from. Here I set a
// plain directory for it, but non-file-system sources are possible too:
cfg.setDirectoryForTemplateLoading(new File("/where/you/store/templates"));

// Set the preferred charset template files are stored in. UTF-8 is
// a good choice in most applications:
cfg.setDefaultEncoding("UTF-8");

// Sets how errors will appear.
// During web page *development* TemplateExceptionHandler.HTML_DEBUG_HANDLER is better.
cfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
```
从现在开始，应该使用单实例配置(也就是说，它是单例的)。 请注意不管一个系统有多少独立的组件来使用 FreeMarker， 它们都会使用他们自己私有的 Configuration 实例。

> Warning!
> 不需要重复创建 Configuration 实例； 它的代价很高，尤其是会丢失缓存。Configuration 实例就是应用级别的单例。

当使用多线程应用程序(比如Web网站)，Configuration 实例中的设置就不能被修改。它们可以被视作为 "有效的不可改变的" 对象， 也可以继续使用 安全发布 技术 (参考 JSR 133 和相关的文献)来保证实例对其它线程也可用。比如， 通过final或volatile字段来声明实例，或者通过线程安全的IoC容器，但不能作为普通字段。 (Configuration 中不处理修改设置的方法是线程安全的。)

### 创建数据模型

你可以使用 java.lang 和 java.util 包中的类， 还有用户自定义的Java Bean来构建数据对象：

- 使用 java.lang.String 来构建字符串。
- 使用 java.lang.Number 来派生数字类型。
- 使用 java.lang.Boolean 来构建布尔值。
- 使用 java.util.List 或Java数组来构建序列。
- 使用 java.util.Map 来构建哈希表。

使用自定义的bean类来构建哈希表，bean中的项和bean的属性对应。比如， product 的 price 属性 (getProperty())可以通过 product.price 获取。


我们为 模板开发指南部分演示的第一个例子 来构建数据模型。为了方便说明，这里再展示一次示例：

```
(root)
  |
  +- user = "Big Joe"
  |
  +- latestProduct
      |
      +- url = "products/greenmouse.html"
      |
      +- name = "green mouse"
```

下面是构建这个数据模型的Java代码片段：


```
// Create the root hash
Map<String, Object> root = new HashMap<>();
// Put string ``user'' into the root
root.put("user", "Big Joe");
// Create the hash for ``latestProduct''
Map<String, Object> latest = new HashMap<>();
// and put it into the root
root.put("latestProduct", latest);
// put ``url'' and ``name'' into latest
latest.put("url", "products/greenmouse.html");
latest.put("name", "green mouse");
```

在真实应用系统中，通常会使用应用程序指定的类来代替 Map， 它会有JavaBean规范规定的 getXxx/isXxx 方法。比如有一个和下面类似的类：


```
public class Product {

    private String url;
    private String name;
    ...
    
    // As per the JavaBeans spec., this defines the "url" bean property
    public String getUrl() {
        return url;
    }
    
    // As per the JavaBean spec., this defines the "name" bean property
    public String getName() {
        return name;
    }
    
    ...
    
}
```

将它的实例放入数据模型中，就像下面这样：

```
Product latestProducts = getLatestProductFromDatabaseOrSomething();
root.put("latestProduct", latestProduct);
```

如果latestProduct 是 Map类型， 模板就可以是相同的，比如 ${latestProduct.name} 在两种情况下都好用。

根root本身也无需是 Map，只要是有 getUser() 和 getLastestProduct() 方法的对象即可。

### 获取模板

模板代表了 freemarker.template.Template 实例。典型的做法是从 Configuration 实例中获取一个 Template 实例。无论什么时候你需要一个模板实例， 都可以使用它的 getTemplate 方法来获取。在 之前 设置的目录中的 test.ftl 文件中存储 示例模板，那么就可以这样来做：


```
Template temp = cfg.getTemplate("test.ftl");
```

当调用这个方法的时候，将会创建一个 test.ftl 的 Template 实例，通过读取 /where/you/store/templates/test.ftl 文件，之后解析(编译)它。Template 实例以解析后的形式存储模板， 而不是以源文件的文本形式。

Configuration 缓存 Template 实例，当再次获得 test.ftl 的时候，它可能再读取和解析模板文件了， 而只是返回第一次的 Template 实例。

### 合并模板和数据模型
我们已经知道，数据模型+模板=输出，我们有了一个数据模型 (root) 和一个模板 (temp)， 为了得到输出就需要合并它们。这是**由模板的 process 方法完成的。它用数据模型root和 Writer 对象作为参数，然后向 Writer 对象写入产生的内容。** 为简单起见，这里我们只做标准的输出：


```
Writer out = new OutputStreamWriter(System.out);
temp.process(root, out);
```

这会向你的终端输出你在模板开发指南部分的 第一个示例 中看到的内容。

**Java I/O 相关注意事项：基于 out 对象，必须保证 out.close() 最后被调用。当 out 对象被打开并将模板的输出写入文件时，这是很电影的做法。其它时候， 比如典型的Web应用程序，那就 不能 关闭 out 对象。FreeMarker 会在模板执行成功后 (也可以在 Configuration 中禁用) 调用 out.flush()，所以不必为此担心。**

请注意，一旦获得了 Template 实例， 就能将它和不同的数据模型进行不限次数 (Template实例是无状态的)的合并。此外， 当 Template 实例创建之后 test.ftl 文件才能访问，而不是在调用处理方法时。

### 将代码放在一起

```
mport freemarker.template.*;
import java.util.*;
import java.io.*;

public class Test {

    public static void main(String[] args) throws Exception {
        
        /* ------------------------------------------------------------------------ */    
        /* You should do this ONLY ONCE in the whole application life-cycle:        */    
    
        /* Create and adjust the configuration singleton */
        Configuration cfg = new Configuration(Configuration.VERSION_2_3_22);
        cfg.setDirectoryForTemplateLoading(new File("/where/you/store/templates"));
        cfg.setDefaultEncoding("UTF-8");
        cfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW);

        /* ------------------------------------------------------------------------ */    
        /* You usually do these for MULTIPLE TIMES in the application life-cycle:   */    

        /* Create a data-model */
        Map root = new HashMap();
        root.put("user", "Big Joe");
        Map latest = new HashMap();
        root.put("latestProduct", latest);
        latest.put("url", "products/greenmouse.html");
        latest.put("name", "green mouse");

        /* Get the template (uses cache internally) */
        Template temp = cfg.getTemplate("test.ftl");

        /* Merge data-model with template */
        Writer out = new OutputStreamWriter(System.out);
        temp.process(root, out);
        // Note: Depending on what `out` is, you may need to call `out.close()`.
        // This is usually the case for file output, but not for servlet output.
    }
}
```
## 数据模型

- 基本内容
- 标量
- 容器
- 方法
- 指令
- 结点变量
- 对象包装

### 基本内容
在 入门 章节中， 我们已经知道如何使用基本的Java类(Map, String,等)来构建数据模型了。在内部，模板中可用的变量都是实现了 freemarker.template.TemplateModel 接口的Java对象。 但在数据模型中，可以使用基本的Java集合类作为变量，因为这些变量会在内部被替换为适当的 TemplateModel 类型。这种功能特性被称作是 对象包装。对象包装功能可以透明地把 任何类型的对象转换为实现了 TemplateModel 接口类型的实例。这就使得下面的转换成为可能，如在模板中把 java.sql.ResultSet 转换为序列变量， 把 javax.servlet.ServletRequest 对象转换成包含请求属性的哈希表变量， 甚至可以遍历XML文档作为FTL变量(参考这里)。包装(转换)这些对象， 需要使用合适的，也就是所谓的 对象包装器 实现(可能是自定义的实现)； 这将在后面讨论。 现在的要点是想从模板访问任何对象，它们早晚都要转换为实现了 TemplateModel 接口的对象。那么首先你应该熟悉来写 TemplateModel 接口的实现类。

有一个粗略的 freemarker.template.TemplateModel 子接口对应每种基本变量类型(TemplateHashModel 对应哈希表， TemplateSequenceModel 对应序列， TemplateNumberModel 对应数字等等)。比如，想为模板使用 java.sql.ResultSet 变量作为一个序列，那么就需要编写一个 TemplateSequenceModel 的实现类，这个类要能够读取 java.sql.ResultSet 中的内容。我们常这么说，使用 TemplateModel 的实现类 包装 了 java.sql.ResultSet，基本上只是封装 java.sql.ResultSet 来提供使用普通的 TemplateSequenceModel 接口访问它。请注意一个类可以实现多个 TemplateModel 接口；这就是为什么FTL变量可以有多种类型 (参看 模板开发指南/数值，类型/基本内容)

请注意，这些接口的一个细小的实现是和 freemarker.template 包一起提供的。 例如，将一个 String 转换成FTL的字符串变量， 可以使用 SimpleScalar，将 java.util.Map 转换成FTL的哈希表变量，可以使用 SimpleHash 等等。

如果想尝试自己的 TemplateModel 实现， 一个简单的方式是创建它的实例，然后将这个实例放入数据模型中 (也就是把它 放到 哈希表的根root上)。 对象包装器将会给模板提供它的原状，因为它已经实现了 TemplateModel 接口，所以没有转换(包装)的需要。 (这个技巧当你不想用对象包装器来包装(转换)某些对象时仍然有用。)
### 标量
有4种类型的标量：

- 布尔值
- 数字
- 字符串
- 日期类型(子类型： 日期(没有时间部分)，时间或者日期-时间)
 
### 容器
- 哈希表
- 序列
- 集合

容器包括哈希表，序列和集合三种类型。

#### 哈希表
哈希表是实现了 TemplateHashModel 接口的Java对象。TemplateHashModel 有两个方法： TemplateModel get(String key)，这个方法根据给定的名称返回子变量， boolean isEmpty()，这个方法表明哈希表是否含有子变量。 get 方法当在给定的名称没有找到子变量时返回null。

TemplateHashModelEx 接口扩展了 TemplateHashModel。它增加了更多的方法，使得可以使用内建函数 values 和 keys 来枚举哈希表中的子变量。

经常使用的实现类是 **SimpleHash**，该类实现了 TemplateHashModelEx 接口。从内部来说，它使用一个 java.util.Hash 类型的对象存储子变量。 SimpleHash 类的方法可以添加和移除子变量。 这些方法应该用来在变量被创建之后直接初始化。

在FTL中，容器是一成不变的。那就是说你不能添加，替换和移除容器中的子变量。

#### 序列
序列是实现了 TemplateSequenceModel 接口的Java对象。它包含两个方法：TemplateModel get(int index) 和 int size()。

经常使用的实现类是 **SimpleSequence**。该类内部使用一个 java.util.List 类型的对象存储它的子变量。 SimpleSequence 有添加子元素的方法。 在序列创建之后应该使用这些方法来填充序列。

#### 集合
集合是实现了 TemplateCollectionModel 接口的Java对象。这个接口定义了一个方法： TemplateModelIterator iterator()。 TemplateModelIterator 接口和 java.util.Iterator 相似，但是它返回 TemplateModels 而不是 Object， 而且它能抛出 TemplateModelException 异常。

通常使用的实现类是 **SimpleCollection**。

### 方法
### 指令
### 结点变量
### 对象包装

# 模板语言参考
