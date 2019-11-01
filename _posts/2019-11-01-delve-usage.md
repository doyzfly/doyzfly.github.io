---
layout: post
title: Golang调试工具Delve使用简介
categories: golang
description: Golang调试工具Delve使用简介
keywords: Golang, Delve, Debug
---


Delve 是一款很不错的 Golang 调试工具，可以实现类似 Visual Studio 的断点调试功能，也可以用来在程序 Crash 的时候生成 Coredump 文件，此外 Delve 也适合用于调试 Web Server。

## Delve 项目
[Github链接](https://github.com/go-delve/delve)

## 安装 Delve
```bash
go get -u github.com/go-delve/delve/cmd/dlv
```

## Delve 常用命令
| 命令 | 功能 | 
| :----- | :----- | 
| dlv attach | 后面跟 pid，用来 Debug 编译好的 Golang 程序 |
| dlv core | 用于 coredump |
| dlv debug | 后面跟要调试的 go 文件，进入 Debug |
| dlv test | Debug test 函数 |

## 编写用于 Debug 的 Golang Web Server  
main.go， 写一个 Hello World 的 Web Server
```golang
package main

import (
	"net/http"
)

const endpoint = ":8000"

type helloHandler struct{}

func (h *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello, world!"))
}

func main() {
	http.Handle("/", &helloHandler{})
	http.ListenAndServe(endpoint, nil)
}
```

## 进行断点调试
```bash
➜  test dlv debug ./main.go
Type 'help' for list of commands.
(dlv) help
The following commands are available:
    args ------------------------ Print function arguments.
    break (alias: b) ------------ Sets a breakpoint.
    breakpoints (alias: bp) ----- Print out info for active breakpoints.
    call ------------------------ Resumes process, injecting a function call (EXPERIMENTAL!!!)
    clear ----------------------- Deletes breakpoint.
    clearall -------------------- Deletes multiple breakpoints.
    condition (alias: cond) ----- Set breakpoint condition.
    config ---------------------- Changes configuration parameters.
    continue (alias: c) --------- Run until breakpoint or program termination.
    deferred -------------------- Executes command in the context of a deferred call.
    disassemble (alias: disass) - Disassembler.
    down ------------------------ Move the current frame down.
    edit (alias: ed) ------------ Open where you are in $DELVE_EDITOR or $EDITOR
    exit (alias: quit | q) ------ Exit the debugger.
    frame ----------------------- Set the current frame, or execute command on a different frame.
    funcs ----------------------- Print list of functions.
    goroutine (alias: gr) ------- Shows or changes current goroutine
    goroutines (alias: grs) ----- List program goroutines.
    help (alias: h) ------------- Prints the help message.
    libraries ------------------- List loaded dynamic libraries
    list (alias: ls | l) -------- Show source code.
    locals ---------------------- Print local variables.
    next (alias: n) ------------- Step over to next source line.
    on -------------------------- Executes a command when a breakpoint is hit.
    print (alias: p) ------------ Evaluate an expression.
    regs ------------------------ Print contents of CPU registers.
    restart (alias: r) ---------- Restart process.
    set ------------------------- Changes the value of a variable.
    source ---------------------- Executes a file containing a list of delve commands
    sources --------------------- Print list of source files.
    stack (alias: bt) ----------- Print stack trace.
    step (alias: s) ------------- Single step through program.
    step-instruction (alias: si)  Single step a single cpu instruction.
    stepout (alias: so) --------- Step out of the current function.
    thread (alias: tr) ---------- Switch to the specified thread.
    threads --------------------- Print out info for every traced thread.
    trace (alias: t) ------------ Set tracepoint.
    types ----------------------- Print list of types
    up -------------------------- Move the current frame up.
    vars ------------------------ Print package variables.
    whatis ---------------------- Prints type of an expression.
Type help followed by a command for full documentation.
```

在这个交换环境下可以进行断点调试，支持的命令主要有：    
| 命令 | alias | 功能 | 
| :-----| :----- | :----- | 
| break | b | 设置断点 |
| continue | c | 运行至断点 |
| print | p | 打印变量 |
| funcs |  | 显示支持的函数 |

### 查看包名/函数名包含 main 的函数  
```bash
funcs main
```

```bash
(dlv) funcs main
crypto/x509.domainToReverseLabels
crypto/x509.matchDomainConstraint
internal/x/net/http/httpproxy.(*domainMatch).match
internal/x/net/http/httpproxy.domainMatch.match
main.(*helloHandler).ServeHTTP
main.init
main.main
net.absDomainName
net.isDomainName
net/http.(*body).bodyRemains
net/http.requestBodyRemains
runtime.main
runtime.main.func1
runtime.main.func2
type..eq.internal/x/net/http/httpproxy.domainMatch
type..hash.internal/x/net/http/httpproxy.domainMatch
```
  
funcs [param] 默认是返回包含 param 的func，可以看到 main package 包含3个 func  
```bash
main.(*helloHandler).ServeHTTP
main.init
main.main
```

### 设置断点  
```bash
break [name] <linespec>
```

1. 在 main 处设置一个断点，break 的参数可以是 funcs 查看等到的 func name  
```bash
break main.main
```
  
```bash
(dlv) break main.main
Breakpoint 1 set at 0x1331513 for main.main() ./main.go:15
```
  
2. 设置断点的另外一种方式  
```bash
<filename>:<line>  
```
  
```bash
(dlv) break ./main.go:15
Breakpoint 1 set at 0x1331513 for main.main() ./main.go:15
```

3. linespec 支持的完整形式
[linespec说明](https://github.com/go-delve/delve/blob/master/Documentation/cli/locspec.md)  

```bash
*<address> Specifies the location of memory address address. address can be specified as a decimal, hexadecimal or octal number

<filename>:<line> Specifies the line line in filename. filename can be the partial path to a file or even just the base name as long as the expression remains unambiguous.

<line> Specifies the line line in the current file

+<offset> Specifies the line offset lines after the current one

-<offset> Specifies the line offset lines before the current one

<function>[:<line>] Specifies the line line inside function. The full syntax for function is <package>.(*<receiver type>).<function name> however the only required element is the function name, everything else can be omitted as long as the expression remains unambiguous. For setting a breakpoint on an init function (ex: main.init), the <filename>:<line> syntax should be used to break in the correct init function at the correct location.

/<regex>/ Specifies the location of all the functions matching regex
```


### 运行至断点
continue 命令运行至断点  

```bash
continue
```
  
```bash
(dlv) continue
> main.main() ./main.go:15 (hits goroutine(1):1 total:1) (PC: 0x1331513)
    10:
    11:	func (h *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    12:		w.Write([]byte("Hello, world!"))
    13:	}
    14:
=>  15:	func main() {
    16:		http.Handle("/", &helloHandler{})
    17:		http.ListenAndServe(endpoint, nil)
    18:	}
```

### 打印变量
1. 可以继续设置断点至 ServeHTTP 函数，并执行 continue 开启 Web Server 服务  

```bash
(dlv) break main.(*helloHandler).ServeHTTP
Breakpoint 2 set at 0x1331443 for main.(*helloHandler).ServeHTTP() ./main.go:11
(dlv) c
```
  
2. 访问 Web Server
这时候 Web Server 的 Delve 需要输入  
```bash
➜  curl http://localhost:8000
Hello, world!%
```

3. 在执行至断点 ServeHTTP 时，可输入 print 查询变量
- 输入 n 回车，单步执行，
- 输入 args 打印出所有的方法参数信息
- 输入 locals 打印所有的本地变量
  
```bash
(dlv) c
> main.(*helloHandler).ServeHTTP() ./main.go:11 (hits goroutine(26):1 total:13) (PC: 0x1331443)
     6:
     7:	const endpoint = ":8000"
     8:
     9:	type helloHandler struct{}
    10:
=>  11:	func (h *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    12:		w.Write([]byte("Hello, world!"))
    13:	}
    14:
    15:	func main() {
    16:		http.Handle("/", &helloHandler{})
(dlv) n
> main.(*helloHandler).ServeHTTP() ./main.go:12 (PC: 0x1331451)
     7:	const endpoint = ":8000"
     8:
     9:	type helloHandler struct{}
    10:
    11:	func (h *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
=>  12:		w.Write([]byte("Hello, world!"))
    13:	}
    14:
    15:	func main() {
    16:		http.Handle("/", &helloHandler{})
    17:		http.ListenAndServe(endpoint, nil)
(dlv) args
h = (*main.helloHandler)(0x16485d0)
w = net/http.ResponseWriter(*net/http.response) 0xc00010b998
r = ("*net/http.Request")(0xc0001c0200)
(dlv) locals
(no locals)
(dlv) print r.Method
"GET"
(dlv)
```