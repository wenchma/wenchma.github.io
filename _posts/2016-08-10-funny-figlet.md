---
layout: post
title: "Funny Figlet"
categories: tech
tags: linux figlet
date: 2016-08-10 17:06:53
---

```
  __ _       _      _   
 / _(_) __ _| | ___| |_ 
| |_| |/ _` | |/ _ \ __|
|  _| | (_| | |  __/ |_ 
|_| |_|\__, |_|\___|\__|
       |___/            

```

Figlet is a tool that generates the text to ASCII art.
It means Frank, Ian & Glenn's Letters who are the authors.

## Installation

```bash
$ sudo apt-get install figlet
```

or

[Download](https://github.com/cmatsuoka/figlet/archive/2.2.5.tar.gz)

```bash
$ tar -zxvf figlet-2.2.4.tar.gz
$ cd figlet
$ make figlet
```

## Usage

```
$ figlet welcome
              _                          
__      _____| | ___ ___  _ __ ___   ___ 
\ \ /\ / / _ \ |/ __/ _ \| '_ ` _ \ / _ \
 \ V  V /  __/ | (_| (_) | | | | | |  __/
  \_/\_/ \___|_|\___\___/|_| |_| |_|\___|

$ figlet welcome -f fonts/slant.flf 
                __                        
 _      _____  / /________  ____ ___  ___ 
| | /| / / _ \/ / ___/ __ \/ __ `__ \/ _ \
| |/ |/ /  __/ / /__/ /_/ / / / / / /  __/
|__/|__/\___/_/\___/\____/_/ /_/ /_/\___/ 


```

## Reference

* [Figlet Home](http://www.figlet.org)
* [Figlet Github](https://github.com/cmatsuoka/figlet)
