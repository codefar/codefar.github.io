---
title: node-cli之commander
categories:
  - node
comments: true
tags:
  - node
  - cli
description: node
date: 2021-08-27 15:37:20

---

# 简介

完整的 node.js 命令行解决方案。


# 准备

* 新建node工程

```
mkdir commander-demo && cd commander-demo
npm init -y
```

* 安装

```
npm install commander
```

## 声明 program 变量

为简化使用，Commander 提供了一个全局对象。本文档的示例代码均按此方法使用：

```
const { program } = require('commander');
program.version('0.0.1');
```

如果程序较为复杂，用户需要以多种方式来使用 ```Commander```，如单元测试等。创建本地`Command`对象是一种更好的方式：

```
const { Command } = require('commander');
const program = new Command();
program.version('0.0.1');
```

要在 ```ECMAScript``` 模块中使用命名导入，可从`commander/esm.mjs`中导入。

```
// index.mjs
import { Command } from 'commander/esm.mjs';
const program = new Command();
```

TypeScript 用法：

```
// index.ts
import { Command } from 'commander';
const program = new Command();
```

## 选项

Commander 使用`.option()`方法来定义选项，同时可以附加选项的简介。每个选项可以定义一个短选项名称（-后面接单个字符）和一个长选项名称（--后面接一个或多个单词），使用逗号、空格或`|`分隔。

解析后的选项可以通过`Command`对象上的`.opts()`方法获取，同时会被传递给命令处理函数。可以使用`.getOptionValue()`和`.setOptionValue()`操作单个选项的值。

对于多个单词的长选项，选项名会转为驼峰命名法（camel-case），例如`--template-engine`选项可通过`program.opts().templateEngine`获取。

**多个短选项可以合并简写，其中最后一个选项可以附加参数**。 例如，`-a -b -p 80`也可以写为`-ab -p80`，甚至进一步简化为`-abp80`。

`--`可以标记选项的结束，后续的参数均不会被命令解释，可以正常使用。

默认情况下，选项在命令行中的顺序不固定，一个选项可以在其他参数之前或之后指定。

### 常用选项类型，boolean 型选项和带参数选项

有两种最常用的选项，

* 一类是 ```boolean``` 型选项，选项无需配置参数，
* 另一类选项则可以设置参数（使用尖括号声明在该选项后，如`--expect <value>`）。如果在命令行中不指定具体的选项及参数，则会被定义为`undefined`。

示例代码：[options-common.js](https://github.com/tj/commander.js/blob/master/examples/options-common.js)

```
program
  .option('-d, --debug', 'output extra debugging')
  .option('-s, --small', 'small pizza size')
  .option('-p, --pizza-type <type>', 'flavour of pizza');

program.parse(process.argv);

const options = program.opts();
if (options.debug) console.log(options);
console.log('pizza details:');
if (options.small) console.log('- small pizza size');
if (options.pizzaType) console.log(`- ${options.pizzaType}`);


$ pizza-options -p
error: option '-p, --pizza-type <type>' argument missing

$ pizza-options -d -s -p vegetarian
{ debug: true, small: true, pizzaType: 'vegetarian' }
pizza details:
- small pizza size
- vegetarian

$ pizza-options --pizza-type=cheese
pizza details:
- cheese
```

通过`program.parse(arguments)`方法处理参数，没有被使用的选项会存放在`program.args`数组中。该方法的参数是可选的，默认值为`process.argv`。

### 选项的默认值

选项可以设置一个默认值。

示例代码：[options-defaults.js](https://github.com/tj/commander.js/blob/master/examples/options-defaults.js)

```
program
  .option('-c, --cheese <type>', 'add the specified type of cheese', 'blue');

program.parse();

console.log(`cheese: ${program.opts().cheese}`);
$ pizza-options
cheese: blue
$ pizza-options --cheese stilton
cheese: stilton
```

### 必填选项

通过`.requiredOption()`方法可以设置选项为必填。必填选项要么设有默认值，要么必须在命令行中输入，对应的属性字段在解析时必定会有赋值。该方法其余参数与`.option()`一致。

示例代码：[options-required.js](https://github.com/tj/commander.js/blob/master/examples/options-required.js)

```
program
  .requiredOption('-c, --cheese <type>', 'pizza must have cheese');

program.parse();
$ pizza
error: required option '-c, --cheese <type>' not specified
```

### 版本选项

`.version()`方法可以设置版本，其默认选项为`-V`和`--version`，设置了版本后，命令行会输出当前的版本号。

```
program.version('0.0.1');
$ ./examples/pizza -V
0.0.1
```

版本选项也支持自定义设置选项名称，可以在`.version()`方法里再传递一些参数（长选项名称、描述信息），用法与`.option()`方法类似。

```
program.version('0.0.1', '-v, --vers', 'output the current version');
```

## 命令

通过`.command()`或`.addCommand()`可以配置命令，有两种实现方式：为命令绑定处理函数，或者将命令单独写成一个可执行文件（详述见后文）。子命令支持嵌套（[示例代码](https://github.com/tj/commander.js/blob/master/examples/nestedCommands.js)）。

`.command()`的第一个参数为命令名称。命令参数可以跟在名称后面，也可以用`.argument()`单独指定。参数可为必选的（尖括号表示）、可选的（方括号表示）或变长参数（点号表示，如果使用，只能是最后一个参数）。

使用`.addCommand()`向`program`增加配置好的子命令。

例如:

```
// 通过绑定处理函数实现命令（这里的指令描述为放在`.command`中）
// 返回新生成的命令（即该子命令）以供继续配置
program
  .command('clone <source> [destination]')
  .description('clone a repository into a newly created directory')
  .action((source, destination) => {
    console.log('clone command called');
  });

// 通过独立的的可执行文件实现命令 (注意这里指令描述是作为`.command`的第二个参数)
// 返回最顶层的命令以供继续添加子命令
program
  .command('start <service>', 'start named service')
  .command('stop [service]', 'stop named service, or all if no name supplied');

// 分别装配命令
// 返回最顶层的命令以供继续添加子命令
program
  .addCommand(build.makeBuildCommand());
```

使用`.command()`和`addCommand()`来指定选项的相关设置。当设置`hidden: true`时，该命令不会打印在帮助信息里。当设置`isDefault: true`时，若没有指定其他子命令，则会默认执行这个命令（[样例](https://github.com/tj/commander.js/blob/master/examples/defaultCommand.js)）。

### 命令参数

如上所述，字命令的参数可以通过`.command()`指定。对于有独立可执行文件的子命令来书，参数只能以这种方法指定。而对其他子命令，参数也可用以下方法。

在`Command`对象上使用`.argument()`来按次序指定命令参数。该方法接受参数名称和参数描述。参数可为必选的（尖括号表示，例如`<required>`）或可选的（方括号表示，例如`[optional]`）。

示例代码：[argument.js](https://github.com/tj/commander.js/blob/master/examples/argument.js)

```
program
  .version('0.1.0')
  .argument('<username>', 'user to login')
  .argument('[password]', 'password for user, if required', 'no password given')
  .action((username, password) => {
    console.log('username:', username);
    console.log('password:', password);
  });
```

在参数名后加上`...`来声明可变参数，且只有最后一个参数支持这种用法。可变参数会以数组的形式传递给处理函数。例如：

```
program
  .version('0.1.0')
  .command('rmdir')
  .argument('<dirs...>')
  .action(function (dirs) {
    dirs.forEach((dir) => {
      console.log('rmdir %s', dir);
    });
  })
```

有一种便捷方式可以一次性指定多个参数，但不包含参数描述：

```
program
  .arguments('<username> <password>');
```

### 处理函数

命令处理函数的参数，为该命令声明的所有参数，除此之外还会附加两个额外参数：一个是解析出的选项，另一个则是该命令对象自身。

示例代码：[thank.js](https://github.com/tj/commander.js/blob/master/examples/thank.js)

```
program
  .argument('<name>')
  .option('-t, --title <honorific>', 'title to use before name')
  .option('-d, --debug', 'display some debugging')
  .action((name, options, command) => {
    if (options.debug) {
      console.error('Called %s with options %o', command.name(), options);
    }
    const title = options.title ? `${options.title} ` : '';
    console.log(`Thank-you ${title}${name}`);
  });
```

````
program
  .command('init')
  .description('生成一个模版工程')
  .action(cmd => {
    const opts = cleanArgs(cmd)
    require('./commands/init')(opts)
  })
  
  function cleanArgs(cmd) {
    if (!cmd.options) {
      throw Error(`命令错误, 未能识别的命令: ${cmd}`)
    }

    // hook 记录 terminal 的输出
    statLogsFromStd(`${cmd._name}.log`)

    const args = {}
    cmd.options.forEach(o => {
      const key = o.long
        .replace(/^--/, '')
        .replace(/-(\w)/g, ($0, $1) => $1.toUpperCase()) // 将中横线换成驼峰
      // if an option is not present and Command has a method with the same name
      // it should not be copied
      if (typeof cmd[key] !== 'function') {
        args[key] = cmd[key]
      }
    })
    return args
 }
 
 //commands/init.js
 module.exports = async function () {
  await reportInit.start()
  logger.warn(msg)
  exec('open http://talos.sankuai.com/#/cli/MRN')
  await reportInit.end()
}
````

### 独立的可执行（子）命令

当`.command()`带有描述参数时，就意味着使用独立的可执行文件作为子命令。 Commander 将会尝试在入口脚本（例如`./examples/pm`）的目录中搜索`program-command`形式的可执行文件，例如`pm-install`、`pm-search`。通过配置选项`executableFile`可以自定义名字。

你可以在可执行文件里处理（子）命令的选项，而不必在顶层声明它们。

示例代码：[pm](https://github.com/tj/commander.js/blob/master/examples/pm)

```
program
  .version('0.1.0')
  .command('install [name]', 'install one or more packages')
  .command('search [query]', 'search with optional query')
  .command('update', 'update installed packages', { executableFile: 'myUpdateSubCommand' })
  .command('list', 'list packages installed', { isDefault: true });

program.parse(process.argv);
```

如果该命令需要支持全局安装，请确保有对应的权限，例如`755`

## 自动化帮助信息

帮助信息是 Commander 基于你的程序自动生成的，默认的帮助选项是`-h,--help`。

示例代码：[pizza](https://github.com/tj/commander.js/blob/master/examples/pizza)

```
$ node ./examples/pizza --help
Usage: pizza [options]

An application for pizza ordering

Options:
  -p, --peppers        Add peppers
  -c, --cheese <type>  Add the specified type of cheese (default: "marble")
  -C, --no-cheese      You do not want any cheese
  -h, --help           display help for command
```

如果你的命令中包含了子命令，会默认添加`help`命令，它可以单独使用，也可以与子命令一起使用来提示更多帮助信息。用法与`shell`程序类似：

```
shell help
shell --help

shell help spawn
shell spawn --help
```