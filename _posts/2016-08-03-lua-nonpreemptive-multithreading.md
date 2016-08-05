---
layout: post
title: "Lua Non-Preemptive Multithreading"
categories: tech
tags: lua
date: 2016-08-03 17:15:40
---

## Lua Coroutine

Lua Coroutine is a kind of collaborative multithreading, equivalent to a thread. A pair yield-resume switches control from one thread to another.
However, unlike "real" multithreading, coroutines are non preemptive. A coroutine only suspends its execution by explicitly calling a `yield` function.

### Conroutine Grammar

| Method                              | Description                                                                                                    |
|-------------------------------------|----------------------------------------------------------------------------------------------------------------|
| coroutine.create (f)                | Creates a new coroutine, with f must be a function, Returns this new coroutine, an object with type "thread".  |
| coroutine.resume (co [, val1, ···]) | Starts or continues the execution of coroutine 					                       |
| coroutine.yield (...)               | Suspends the execution of the calling coroutine. Any arguments to yield are passed as extra results to resume. |
| coroutine.running ()                | Returns the running coroutine plus a boolean, true when the running coroutine is the main one.                 |
| coroutine.status (co)               | Returns the status of coroutine co, as a string: "running", "suspended","normal" "dead"                        |
| coroutine.wrap (f)                  | Creates a new coroutine, Returns a function that resumes the coroutine each time it is called.                 |
| coroutine.isyieldable ()            | Returns true when the running coroutine can yield.                                                             |
