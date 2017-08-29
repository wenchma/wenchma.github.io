---
layout: post
title: "Shell变量总结"
categories: tech
tags: shell
date: 2016-11-10 16:55:45
---

Shell 编程有丰富的变量使用规则以实现各种各样的需求，这里做了一下总结，以备以后用着方便，内容主要来源于网络和平时工作经验.

## 1. 特殊符号意义：

* $n - $1 the first parameter,$2 the second...
* $# - The number of command-line parameters.
* $0 - The name of current program.
* $? - Last command or function's return value.
* $$ - The program's PID.
* $! - Last program's PID.
* $@ - Save all the parameters.
* $- - 显示shell使用的当前选项
* $_ - 之前命令的最后一个参数

## 2. 变量引用方式

var="helloworld"
file=/dir1/dir2/dir3/my.file.txt

| variable                 | Description                                                                         |
|--------------------------|-------------------------------------------------------------------------------------|
| $var or ${var}           | return value of var, 注意区别$var_str 和${var}_str                                    |
| ${var:0:5}               | return first 5 characters, result is hello                                          |
| ${var:5}                 | return from start_index 5 to end, result is world                                   |
| ${file#*/}               | 删掉第一个/及其左边的字符串： dir1/dir2/dir3/my.file.txt                                 |
| ${file##*/}              | 删掉最后一个 / 及其左边的字符串：my.file.txt                                             |
| ${file%/*}               | 删掉最后一个 / 及其右边的字符串：/dir1/dir2/dir3                                         |
| ${file%%/*}              | 删掉第一个 / 及其右边的字符串：(空值)                                                    |
| ${file/.}                | delete first . : /dir1/dir2/dir3/myfile.txt                                         |
| ${file/dir/path}         | 将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt                                  |
| ${file//dir/path}        | 将全部dir 替换为 path：/path1/path2/path3/my.file.txt                                 |
| ${var:-newstring}        | if var is empty or undefined, return newsting, if not, reutrn var                   |
| ${var:=newstring}        | if var is empty or undefined, return newsting and assign to var, if not, reutrn var |
| ${var:+newstring}        | if var is empty, return newstring, if not, return empty                             |
| ${var:?newstring}        | if var is empty or undefined, write newstring to stderr, if not, return var         |
| $(command)               | return the stdout of runing command                                                 |
| $((算术表达式))            | $((5+6 * 2)) return 17 , i=5 $((i * 6)) return 30                                   |
| let a=${i} * 6           | i=5, a=30, but a=${i} * 6 return a=5 * 6 						                     |
| array                    |a=(1 2 3 4 5),$a = 1,${a[@或 * ]} = str with items, ${#a[@或 * ]}=len, ${a[index]}    |

## 3. for 循环

* Array

```
$ for ((i=1;i<11;i++));do echo $i;done
1
2
3
4
5
6
7
8
9
10
$ for i in {1..10};do echo $i;done
```

* Map

```
$ declare -A map=(["100"]="1" ["200"]="2")

# output all keys 
$ echo ${!map[@]}
200 100

# output all values
$ echo ${map[@]}  

$ map["300"]="3" 

$ for key in ${!map[@]};do echo $key=${map[$key]};done
200=2
300=3
100=1

```

## 4. if 判断

* -e filename 如果 filename存在，则为真
* -b filename 如果 filename 存在且是一个块特殊文件则为真
* -c filename 如果 filename 存在且是一个字特殊文件则为真
* -d filename 如果 filename为目录，则为真 
* -f filename 如果 filename为常规文件，则为真
* -L filename 如果 filename为符号链接，则为真
* -r filename 如果 filename可读，则为真 
* -w filename 如果 filename可写，则为真 
* -x filename 如果 filename可执行，则为真
* -s filename 如果文件长度不为0，则为真
* -h filename 如果文件是软链接，则为真
* -z string   如果字符串为空，则为真
* -n 字符串不为"null" 
* filename1 -nt filename2 如果 filename1比 filename2新，则为真。
* filename1 -ot filename2 如果 filename1比 filename2旧，则为真。
* -eq 等于
* -ne 不等于
* -gt 大于
* -ge 大于等于
* -lt 小于
* -le 小于等于
* 至于！号那就是取非了呗！