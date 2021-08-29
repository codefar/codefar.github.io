---
title: Linux Shell 常用用法知识点
date: 2016-04-26 18:54:59
tags:
---

# trap
trap是一个shell内建命令，它用来在脚本中指定信号如何处理 .

例如：如果我们使用shell脚本执行build任务，如果中途通过Ctrl+C中断执行。此时我们就需要清理我们编译中出错的文件。

> trap "rm -fr build ;exit 0" SIGINT

 此时，通过以上我们既可以清理掉build文件夹。

# getopts / getopt

    ./work.sh --prefix outDir -f config.xml

getopts和getopt相似,区别是getopt是独立的可执行文件，而getopts是由Bash内置的
