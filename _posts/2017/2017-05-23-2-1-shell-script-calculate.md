---
layout: post
title: Linux Shell 脚本（2）——算术运算
category: 2017年5月
tags: Shell
keywords: Shell
description: 主要介绍 Linux Shell 脚本的算数运算
---

## 说明

　　在[上一篇][shellFundamental]中，我们介绍了`Shell脚本`的基本结构，本篇着重点在于简单的**加减乘除和求余**这几个算术运算。

---

## expr 命令

```shell
#!/bin/bash

### expr 算术运算 ###
a=60
b=20
echo --------expr 命令---------
echo "初始值：a=${a} b=${b}"
echo -n '直接执行：expr ${a} + ${b}: '
expr ${a} + ${b}  # 这里会直接输出 a+b 的结果
echo -n '先执行 c=`expr $a + $b`，再输出 $c：'
c=`expr ${a} + ${b}` # 这里会先执行 a+b，并将结果赋值给 c
echo ${c}

echo -n a+b=
expr ${a} + ${b}
echo -n a-b=
expr ${a} - ${b}
echo -n a*b=
expr ${a} \* ${b} # 乘法运算必须将*使用反斜杠\转义出来
echo -n a/b=
expr ${a} / ${b}
echo -n c%a=
expr ${c} % ${a}

```

　　`expr`命令格式为：`expr ${var1} op ${var2}`。要求`var1`和`var2`都必须是整数，`op`为算术运算符，支持：`+ - * / %`，其中乘法运算需要添加转义字符`\`，每个字段之间都必须由空格隔开。

---

## let 命令

```shell
#!/bin/bash

a=60
b=20
echo --------let 命令---------
echo "初始值：a=$a b=$b"
echo let 命令通常用于运算后直接赋值
let c=$a+$b
echo 'after let c=a+b, c == '$c
let c=$a-$b
echo 'after let c=a-b, c == '$c
## let 命令在执行乘法运算时乘号可以不转义
let c=$a\*$b
echo 'after let c=a\*b, c == '$c
let c=$a*$b
echo 'after let c=a*b, c == '$c
let c=$a/$b
echo 'after let c=a/b, c == '$c
let c=$a+$b
echo -e -n "before: c=$c, a=$a\t"
let c=c%a
echo 'after let c=c%a, c == '$c
```

　　`let`命令格式为：`let result=${var1}op${var2}`或者`let result=var1opvar2`。要求`var1`和`var2`都是整数，否则计算结果不正确。其中参与运算的值可以不使用`${}`括起来，乘法运算符可以直接使用。

---

## $((expression))命令

```shell
#!/bin/bash

a=60
b=20
echo --------$((expression)) 命令---------
echo "初始值：a=$a b=$b"
echo '$((expression)) 命令通常用于运算后直接赋值'
c=$(($a+$b))
echo 'after c=$(($a+$b)), c == '$c
c=$((a-b))
echo 'after c=$((a-b)), c == '$c
c=$((a*b))
echo 'after c=$((a*b)), c == '$c
c=$((a/b))
echo 'after c=$((a/b)), c == '$c
c=$((a+b))
echo -e -n "before: c=$c, a=$a\t"
c=$((c%a))
echo 'after c=$((c%a)), c == '$c
```

　　`$((expression))`是一种比较简洁的算术运算实现，其格式为：`result=$((${var1}op${var2}))`或者`result=$((var1opvar2))`。

---

## 累加计算

```shell
#!/bin/bash

## 判断参数1是否是整数，是的话返回 true，否的话返回 false
function isNumber() {
	if [ $# -eq 1 ]
	then
		if expr $1 + 0 &>/dev/null; then
			return 0
		else
			return 1
		fi
	else
		return 1
	fi	
}

## 验证参数1是否是数字，如果不是的话返回 10，是的话返回该值
function validate() {
	if isNumber $1; then
		return $1
	else
		return 10
	fi
}

## 通过 expr 命令实现累加
function exprAccumulate() {
	validate $1
	num=$?
	index=1
	while [ ${index} -le ${num} ]
	do
		echo ${index}
		index=`expr ${index} + 1`
	done
	return 0
}

## 通过 let 命令实现累加
function letAccumulate() {
	validate $1
	num=$?
	index=1
	until [ ${index} -gt ${num} ]
	do
		echo $index
		let index+=1
	done
	return 0
}

## 通过 $((expression)) 命令实现累加
function dollarAccumulate() {
	validate $1
	num=$?
	index=1
	while [ ${index} -le ${num} ]; do
		echo $index
		index=$((index+1))
	done
}


echo '使用 expr 命令实现 1 累加到 5'
exprAccumulate 5
echo '使用 let 命令实现 1 累加到 6'
letAccumulate 6
echo '使用 $((expression)) 命令实现 1 累加到 7'
dollarAccumulate 7
```

　　本文简单介绍了`Shell脚本`基本算术运算的三种实现，个人推荐使用第三种`$((expression))`。



|                 |
| --------------- |
| 编写日期：2017-05-23 |
| 发布日期：2017-05-23 |

[shellFundamental]: 2-0-shell-script-fundamental.html "跳转到"Linux Shell 脚本（1）——认识 Shell 脚本""

