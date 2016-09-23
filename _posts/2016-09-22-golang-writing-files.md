---
layout: post
title: "Dive into file-writing in Golang"
categories: tech
tags: golang
date: 2016-09-22 18:48:57
---

## Writing Methods

In this post, we will see how to dump string (or bytes) to a file.
Generally, just open a file for writing,
it’s idiomatic to defer a Close immediately after opening a file.
Belows are almost methods to write a file.

### 1. io.WriteString

```golang
func main() {
	var content = []byte("helloworld")

	// Writer 1: io.WriteString
	f1, _ := os.Create("./output1")
	n, err := io.WriteString(f1, string(content))
	if err != nil {
		panic(err)
	}
	fmt.Printf("Writed %d bytes\n", n)
}
```

### 2. ioutil.WriteFile

```golang
func main() {
	if err := ioutil.WriteFile("output2", content, 0644); err != nil {
		panic(err)
	}
	fmt.Println("ioutil.WriteFile: success")
}
```

### 3. File.Write & File.WriteString

```golang
func main() {
	f3, err := os.OpenFile("output3", os.O_APPEND, 0644)
	if err != nil {
		f3, _ = os.Create("output3")
	}
	f3.Write(content)
	f3.WriteString(string(content))
}
```

### 4. bufio.NewWriter

```golang
func main() {
	// Writer 4: bufio.NewWriter
	f4, err := os.OpenFile("output4", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		f4, _ = os.Create("output4")
	}
	w := bufio.NewWriter(f4)
	if _, err := w.Write(content); err != nil {
		panic(err)
	}
	if err := w.Flush(); err != nil {
		panic(err)
	}
	fmt.Println("bufio.NewWriter Write: success")
	f4.Close()

	f4_2, err := os.OpenFile("output4_2", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		f4_2, _ = os.Create("output4_2")
	}
	w = bufio.NewWriter(f4_2)
	if _, err := w.WriteString(string(content)); err != nil {
		panic(err)
	}
	if err := w.Flush(); err != nil {
		panic(err)
	}
	fmt.Println("bufio.NewWriter WriteString: success")
	f4_2.Close()
}
```

## Benchmark Testing

We use golang `testing` package to write benchmarking tests to examine the performance
of the above writing codes. Don't know how to write benchmark tests, refer to this [post][1]

**Example code**:

```golang
package main

import (
	"bufio"
	"io"
	"io/ioutil"
	"os"
	"testing"
)

var content = []byte("Benchmark Test")

func BenchmarkIOWriteString(b *testing.B) {
	f1, _ := os.Create("./output1_test") 
	defer f1.Close()
	for i := 0; i < b.N; i++ {
		io.WriteString(f1, string(content))
	}
}

func BenchmarkIoUtilWriteFile(b *testing.B) {
	for i := 0; i < b.N; i++ {
		ioutil.WriteFile("./output2_test", content, 0644)
	}
}

func BenchmarkFileWrite(b *testing.B) {
	f3, _ := os.Create("output3_test") 
	defer f3.Close()
	for i := 0; i < b.N; i++ {
		f3.Write(content)
	}
}

func BenchmarkFileWriteString(b *testing.B) {
	f3_2, _ := os.Create("output3_2_test") 
	defer f3_2.Close()
	for i := 0; i < b.N; i++ {
		f3_2.WriteString(string(content))
	}
}

func BenchmarkBufioWrite(b *testing.B) {
	 f4, _ := os.Create("output4_test") 
	 defer f4.Close()
	 w := bufio.NewWriter(f4)
	 for i := 0; i < b.N; i++ {
		 w.Write(content)
	}
 	w.Flush()
}

func BenchmarkBufioWriteString(b *testing.B) {
	 f4_2, _ := os.Create("output4_2_test") 
	 defer f4_2.Close()
	 w := bufio.NewWriter(f4_2)
	 for i := 0; i < b.N; i++ {
		 w.WriteString(string(content))
	}
 	w.Flush()
}
```

**Testing Results:

```
BenchmarkIOWriteString-8	 1000000	      1907 ns/op
ok  	command-line-arguments	1.937s

BenchmarkIoUtilWriteFile-8	   10000	    135832 ns/op
ok  	command-line-arguments	1.708s

BenchmarkFileWrite-8      	 1000000	      1658 ns/op
BenchmarkFileWriteString-8	 1000000	      1749 ns/op
ok  	command-line-arguments	4.014s

BenchmarkBufioWrite-8      	50000000	       142 ns/op
BenchmarkBufioWriteString-8	50000000	       157 ns/op
ok  	command-line-arguments	16.099s
```

[1]: http://dave.cheney.net/2013/06/30/how-to-write-benchmarks-in-go