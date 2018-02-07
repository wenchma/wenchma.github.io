---
layout: post
title: "Linux Calulator bc"
categories: tech
tags: linux
date: 2018-02-07 17:52:33
---

## Overview

`bc` command is a linux command line calculator, can do basic mathematical calculations, like `Integer`, `floating` calculations.

```bash
usage: bc [options] [file ...]
  -h  --help         print this usage and exit
  -i  --interactive  force interactive mode
  -l  --mathlib      use the predefined math routines
  -q  --quiet        don't print initial banner
  -s  --standard     non-standard bc constructs are errors
  -w  --warn         warn about non-standard bc constructs
  -v  --version      print version information and exit
```

The `bc` command supports the following features:
1. Arithmetic operators
2. Increment or Decrement operators
3. Assignment operators
4. Comparison or Relational operators
5. Logical or Boolean operators
6. Math functions
7. Conditional statements
8. Iterative statements

## Usage

### 1. Arithmetic operators

```
$ a=2
$ b=10
$ echo "$a+$b" | bc
12
$ echo "$a-$b" | bc
-8
$ echo "$a*$b" | bc
20
$ echo "$a/$b" | bc
0
$ echo "$a^$b" | bc
1024
$ echo "sqrt(10)" | bc
3
$ echo "scale=2;sqrt(10)" | bc
3.16
```

`bc` 可以用 `scale` 保留小数位, only supports the `/`, `sqrt`, if you want to use it as follows:
```
$ echo "scale=2;3.333+4.444" | bc
7.777
$ echo "scale=2;(3.333+4.444)/1" | bc
7.77

```

From above, it not support math rounding, there is a workaround as follows:
```
$ echo "scale=2;(7.777+0.005)/1" | bc
7.78

```

### 2. Assignment Operators

* var = value : Assign the vale to the variable
* var += value : similar to var = var + value
* var -= value : similar to var = var – value
* var *= value : similar to var = var * value
* var /= value : similar to var = var / value
* var ^= value : similar to var = var ^ value
* var %= value : similar to var = var % value

```
$ echo "var=10;var+=9;var" | bc
19

```

### 3. Increment and Decrement Operators

```
$ echo "var=10;var++" | bc
10
$ echo "var=10;++var" | bc
11
$ echo "var=10;var--" | bc
10
$ echo "var=10;--var" | bc
9

```

### 4. Comparison or Relational Operators

These are used to compare 2 numbers. If the comparison is true, then result is 1. Otherwise(false), returns 0.

Operators: **<**,**<=**,**>**,**>=**,**==**,**!=**.

```
$ echo "10>=10.0" | bc
1
$ echo "10!=10.0" | bc
0

```

### 5. Logical or Boolean Operators

```
$ echo "10 && 5" | bc
1
$ echo "0 || 0" | bc
0
$ echo "!0" | bc
bash: !0: event not found
$ echo "! 0" | bc
1

```

### 6.Mathematical Functions

```
$ echo "length(1.2345)" | bc -l
5
$ echo "scale(1.2345)" | bc -l
4

```

进制转换

```
$ echo "ibase=10;obase=2;15" | bc -l
1111
$ echo "ibase=2;1111" | bc
15

```