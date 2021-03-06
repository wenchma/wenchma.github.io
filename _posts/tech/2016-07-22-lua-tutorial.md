---
layout: post
title:  "Lua Tutorial"
date:   2016-07-22 09:40:42
categories: tech
tags: lua
---

## What is Lua ?

![](/img/lua.png)

Lua is a powerful, efficient, lightweight, embeddable scripting language, 用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。
Lua 是巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组，由Roberto Ierusalimschy、Waldemar Celes 和 Luiz Henrique de Figueiredo所组成并于1993年开发。 

Lua supports procedural programming, object-oriented programming, functional programming, data-driven programming, and data description.

## Installation

```
curl -R -O http://www.lua.org/ftp/lua-5.3.3.tar.gz
tar zxf lua-5.3.3.tar.gz
cd lua-5.3.3
# build Lua
make linux

# test your building
make test

# if ok
$ sudo make install
cd src && mkdir -p /usr/local/bin /usr/local/include /usr/local/lib /usr/local/man/man1 /usr/local/share/lua/5.1 /usr/local/lib/lua/5.1
cd src && install -p -m 0755 lua luac /usr/local/bin
cd src && install -p -m 0644 lua.h luaconf.h lualib.h lauxlib.h ../etc/lua.hpp /usr/local/include
cd src && install -p -m 0644 liblua.a /usr/local/lib
cd doc && install -p -m 0644 lua.1 luac.1 /usr/local/man/man1
$ lua -v
Lua 5.1.5  Copyright (C) 1994-2012 Lua.org, PUC-Rio
```

You can also install lua pkg management tool (like python pip, ruby gem):

```
$ sudo apt-get install luarocks
$ sudo luarocks install luasocket
Installing http://luarocks.org/repositories/rocks/luasocket-3.0rc1-2.src.rock...
Using http://luarocks.org/repositories/rocks/luasocket-3.0rc1-2.src.rock... switching to 'build' mode
Archive:  /tmp/luarocks_luarocks-rock-luasocket-3.0rc1-2-1445/luasocket-3.0rc1-2.src.rock
  inflating: luasocket-3.0rc1-2.rockspec  
 extracting: v3.0-rc1.zip            
Archive:  v3.0-rc1.zip
```

> Note: `make xxx`, where xxx is your platform name. eg. linux, macosx, freebsd...


## Lua Tutorial

### Lua Hello World

1. Interactive mode:

```
$ lua
Lua 5.1.5  Copyright (C) 1994-2012 Lua.org, PUC-Rio
> print("hello".." world")
hello world
>
```

2. Lua scripts:

```
#!/usr/local/bin/lua

print("Hello".." World！")

$ ./test.lua 
Hello World！

$ lua test.lua 
Hello World！
```

### Example: Http Request

```
#!/usr/bin/env lua
local http=require("socket.http")
local ltn12 = require("ltn12")

local request_body = [[login=admin&password=pass]]
local response_body = {}
local path = "/v1/users"
local url = "http://example.com"
local token = ""

local res, code, response_headers = http.request{
	url = string.format("%s/%s", url, path),
	method = "GET",
	headers =
	  {
	  	["Content-Type"] = "application/x-www-form-urlencoded";
	  },
	  source = ltn12.source.string(request_body),
	  sink = ltn12.sink.table(response_body),
}

print(res)
print(code)

if type(response_headers) == "table" then
  for k, v in pairs(response_headers) do
    print(k, v)
  end
end

print("Response body:")
if type(response_body) == "table" then
    print(table.concat(response_body))
  else
    print("Not a table:", type(response_body))
  end
```

> Note: If for https request, you can use Lua ssl lib. `require("ssl.https")`.
  First, install Lua package manager: `sudo apt-get install luarocks`

### Trouble Shooting

If :

```
# luarocks install luasec OPENSSL_LIBDIR=/usr/lib/x86_64-linux-gnu/
Warning: Failed searching manifest: Failed extracting manifest file
Installing https://raw.githubusercontent.com/rocks-moonscript-org/moonrocks-mirror/master/luasec-0.6-1.rockspec...
Using https://raw.githubusercontent.com/rocks-moonscript-org/moonrocks-mirror/master/luasec-0.6-1.rockspec... switching to 'build' mode

Error: Could not find expected file openssl/ssl.h, or openssl/ssl.h for OPENSSL -- you may have to install OPENSSL in your system and/or pass OPENSSL_DIR
or OPENSSL_INCDIR to the luarocks command. Example: luarocks install luasec OPENSSL_DIR=/usr/local
```

It means to install `openssl` lib first. To install openssl:
$ sudo apt-get install  openssl libssl-dev.
then,
`sudo luarocks install luasec OPENSSL_LIBDIR=/usr/lib/x86_64-linux-gnu/` or `sudo luarocks install luasec`.


### Reference

* [Lua 5.3 Manual](http://www.lua.org/manual/5.3/)
* [Lua Tutorial](http://www.runoob.com/lua/lua-tutorial.html)