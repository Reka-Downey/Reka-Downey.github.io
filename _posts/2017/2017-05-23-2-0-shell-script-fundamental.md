---
layout: post
title: Linux Shell 脚本（1）——认识 Shell 脚本
category: 2017年5月
tags: Shell
keywords: Shell
description: 主要介绍 Linux Shell 脚本基础
---



## 执行 Shell 脚本

　　`Shell`脚本可以通过两种方式执行：

- `/bin/bash /path/to/shell/script`通过`/bin/bash`解释器直接执行脚本。
- `chmod +x /path/to/shell/script`授予可执行权限之后直接使用`/path/to/shell/script`命令即可执行该脚本

　　**注意**：`Linux`对`Shell脚本`的后缀名并没有特殊性要求，使用`.sh`作为后缀只是为了方便识别`Shell脚本`。

---

## Shell 脚本结构

### Shell 类型说明

　　`Shell脚本`第一行通常用来指明`Shell脚本`的解析器路径，一般都是采用`/bin/bash`作为脚本解释器，格式为：`#!/path/to/shell/interpreter`。

### Shell 注释

　　`Shell脚本`注释采用`#`作为注释标识符。

### Shell 函数

　　`Shell脚本`函数定义格式为：

```shell
function ${functionName}(){
  # function body
}
```

　　**注意**：由于`Shell脚本`是解释型语言，因此函数必须在函数使用之前定义。

### Shell 主过程

　　`Shell主过程`指可以调用`Shell`基础语法和自定义函数的脚本语句。

---

## Shell 脚本实例

```shell
#!/bin/bash

# 定义 sayHello 函数
function sayHello() {
    if [ $# -eq 1 -a -n "$1" ]; then
        echo "Hello $1"
        return 0
    else
        echo "Hello World"
        return 1
    fi
}

# Shell 主过程
echo -n "Your name: "
read who >/dev/null 2>&1
sayHello ${who}
exit 0
```

　　该实例首先读取客户端输入的字符串，接着将输入的字符串传递给`sayHello`函数，函数中执行参数判断，如果发现参数只有一个并且参数非空的时候，输出 `Hello ${yourName}`，否则输出`Hello World`。



|                 |
| --------------- |
| 编写日期：2017-05-23 |
| 发布日期：2017-05-23 |