# Go语言调试

## delve (dlv)

### dlv简介

Delve是一个用于Go程序的源码级调试器。Delve使你能够通过控制程序的执行、评估变量以及提供线程/`goroutine`状态、CPU寄存器状态等信息与你的程序进行交互。

调试时可以使用`--`向你调试的程序传递可选标识，例如：

```bash
$ dlv exec ./hello -- server --config conf/config.toml
```

**可用命令**

- [dlv attach](#dlv attach) - 附加到正在运行的进程并开始调试。

- [dlv connect](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_connect.md) - 连接至`headless debug server`。

- [dlv core](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_core.md) - 检查coredump文件。

- [dlv dap](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_dap.md) - [实验]启动一个TCP服务器并通过DAP协议（Debug Adaptor Protocol）通信。

- [dlv debug](#dlv debug) - 编译并开始调当前目录下的main包或指定的包。

- [dlv exec](#dlv exec) - 执行可执行文件并开始调试。

- [dlv replay](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_replay.md) - 回放一个rr跟踪。

- [dlv run](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_run.md) - 废弃的命令，使用`debug`代替。

- [dlv test](#dlv test) - 编译测试二进制文件并开始调试程序。

- [dlv trace](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_trace.md) - 编译并开始追踪程序。

- [dlv version](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_version.md) - 打印版本。

- [dlv log](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_log.md) - Help about logging flags

- [dlv backend](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv_backend.md) - Help about the `--backend` flag

- dlv redirect - Help about file redirection

  使用“`dlv [command] --help`”查找关于这个命令更多的信息。

**全局选项**

```
--accept-multiclient               	允许一个headless server支持多个客户端连接
--allow-non-terminal-interactive   	允许没有终端的Delve的交互式会话作为stdin、stdout和stderr。
--api-version int                  	选择headless时的API版本。新客户端应该使用v2。可以通过RPCServer.SetApiVersion重置。参见Documentation/api/json-rpc/README.md。(默认为1)
--backend string                   	选择后端 (see 'dlv help backend'). (default "default")
--build-flags string               	传给编译器构建标志，例如: --build-flags="-tags=integration -mod=vendor -cover -v"
--check-go-version                 	检查正在使用的Go的版本是否与Delve兼容。(默认为true)
--disable-aslr                     	禁用地址空间布局随机化（Address Space Layout Randomization）
--headless                         	只运行调试服务器，在headless模式下
--init string                      	启动文件，由终端客户端执行
-l, --listen string                 调试服务器监听地址(默认为 "127.0.0.1:0")
--log                              	启用调试服务器日志
--log-dest string                  	将日志写到指定的文件或文件描述符中（见'dlv help log'）
--log-output string                	设置输出调试信息的组件，用逗号分隔（见’dlv help log‘）
--only-same-user                   	只允许来自启动此Delve实例的同一用户的连接(默认为true)
-r, --redirect stringArray          为目标进程指定重定向规则（见'dlv help redirect'）
--wd string      				   	运行该程序的工作目录
```

#### dlv attach

这个命令将使Delve控制一个已经运行的进程，并开始一个新的调试会话。当退出调试会话时，你可以选择让该进程继续运行或杀死它。

```
dlv attach pid [executable]
```

**选项**

```
--continue   在启动时继续进行调试进程
```

#### dlv exec

这个命令将使Delve执行二进制文件，并立即附加到进程开始一个新的调试会话。请注意，如果二进制文件在编译时没有禁用优化功能，可能很难正确地调试它。请考虑在Go 1.10或更高版本上用-gcflags="all=-N -l "编译调试二进制文件，在Go的早期版本上用-gcflags="-N -l"。

```
dlv exec <path/to/binary>
```

**选项**

```
--continue     在启动时继续进行调试进程
--tty string   指定目标程序要使用的tty
```

#### dlv debug

在禁用优化的情况下编译你的程序，启动并附加到它。默认情况下，在没有参数的情况下，Delve将编译当前目录下的 "main "包，并开始调试它。或者，你可以指定一个包的名字，Delve将编译该包，并开始一个新的调试会话。

```
dlv debug [package]
```

**选项**

```
--continue        在启动时继续进行调试进程
--output string   二进制文件的输出路径(default "./__debug_bin")
--tty string      指定目标程序要使用的tty
```

#### dlv test

编译一个禁用优化的测试二进制文件，并开始一个新的调试会话。

test命令允许你在单元测试的背景下开始一个新的调试会话。默认情况下，Delve将调试当前目录下的测试程序。另外，你可以指定一个包的名称，Delve将在该包中调试测试程序。双破折号`--`可以用来传递参数给测试程序。

```
dlv test [package] -- -test.v -other-argument
```

另见：“go help testflag”。

```
dlv test [package]
```

**选项**

```
--output string   二进制文件的输出路径(default "debug.test")
```

### 开始使用

#### 调试main包

我们要探讨的第一个CLI子命令是`debug`。如果你和你的`main'包在同一个目录下，这个子命令可以不需要参数运行。
否则，它需要指定包的路径。例如，基于下面这个项目的布局：

```
.
├── github.com/me/foo
├── cmd
│   └── foo
│       └── main.go
├── pkg
│   └── baz
│       ├── bar.go
│       └── bar_test.go
```

如果你当前是在目录`github.com/me/foo/cmd/foo`中，那么你可以直接在命令行执行`dlv debug`。在其他任何位置，比如项目根目录，你需要提供包名：`dlv debug github.com/me/foo/cmd/foo`。要向你的程序传递标志用`--`分开：`dlv debug github.com/me/foo/cmd/foo -- --arg1 value`。

调用该命令将使Delve以最适合调试的方式编译程序。然后，它将执行并附加到程序上，并开始一个调试会话。现在，当调试会话开始时，你就处于程序初始化的最开始。为了进入一个更有用的地方你会想设置一个或两个断点，然后继续执行到该断点。

例如，继续执行到你的程序的`main`函数。

```bash
$ dlv debug github.com/me/foo/cmd/foo
Type 'help' for list of commands.
(dlv) break main.main
Breakpoint 1 set at 0x49ecf3 for main.main() ./test.go:5
(dlv) continue
> main.main() ./test.go:5 (hits goroutine(1):1 total:1) (PC: 0x49ecf3)
     1:	package main
     2:	
     3:	import "fmt"
     4:	
=>   5:	func main() {
     6:		fmt.Println("delve test")
     7:	}
(dlv) 
```

#### 调试测试程序

考虑到与上面相同的目录结构，你可以通过执行你的测试套件来调试你的代码。为此，你可以使用`dlv test`子命令，它需要与`dlv debug`一样的可选包路径，如果不传递任何参数，则会构建当前目录的包。

```bash
$ dlv test github.com/me/foo/pkg/baz
Type 'help' for list of commands.
(dlv) funcs test.Test*
/home/me/go/src/github.com/me/foo/pkg/baz/test.TestHi
(dlv) break TestHi
Breakpoint 1 set at 0x536513 for /home/me/go/src/github.com/me/foo/pkg/baz/test.TestHi() ./test_test.go:5
(dlv) continue
> /home/me/go/src/github.com/me/foo/pkg/baz/test.TestHi() ./bar_test.go:5 (hits goroutine(5):1 total:1) (PC: 0x536513)
     1:	package baz
     2:	
     3:	import "testing"
     4:	
=>   5:	func TestHi(t *testing.T) {
     6:		t.Fatal("implement me!")
     7:	}
(dlv) 
```

正如你所看到的，我们开始调试测试二进制，通过`funcs`命令找到我们的测试函数。`funcs`命令找到了我们的测试函数，该命令使用一个regexp来过滤函数列表，设置一个断点，然后继续执行，直到我们命中这个断点。

可以使用`dlv help`获取更多关于子命令的信息，一旦进入调试会话，你可以通过键入`help`看到所有可用的命令。
就可以看到所有可用的命令。

#### 调试编译器

### 配置和历史命令

如果`$XDG_CONFIG_HOME`被设置，那么配置和命令历史文件就位于`$XDG_CONFIG_HOME/dlv`。否则，它们在Linux上位于`$HOME/.config/dlv`，在其他系统上位于`$HOME/.dlv`。

配置文件`config.yml`包含所有可配置的选项和它们的默认值。命令历史被保存在`.dbg_history`中。

### 调试命令

#### 运行程序

| Command                               | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| [call](#call)                         | 恢复进程，注入一个函数调用（实验性的！！！）                 |
| [continue](#continue)                 | 运行到断点或程序结束                                         |
| [next](#next)                         | 跳转到下一个代码行                                           |
| [rebuild](#rebuild)                   | 重新构建目标可执行文件并重新启动它。如果该可执行文件不是由delve构建的，它就不起作用 |
| [restart](#restart)                   | 重启进程                                                     |
| [step](#step)                         | 单步调试                                                     |
| [step-instruction](#step-instruction) | 单步调试CPU指令                                              |
| [stepout](#stepout)                   | 跳出当前函数                                                 |

#### 操作断点

| Command                     | Description                      |
| --------------------------- | -------------------------------- |
| [break](#break)             | 设置断点                         |
| [breakpoints](#breakpoints) | 打印出生效断点的信息             |
| [clear](#clear)             | 删除断点                         |
| [clearall](#clearall)       | 删除多个断点                     |
| [condition](#condition)     | 设置断点条件                     |
| [on](#on)                   | 当一个断点被击中时，执行一个命令 |
| [toggle](#toggle)           | 打开或关闭一个断点               |
| [trace](#trace)             | 设置跟踪点                       |
| [watch](#watch)             | 设置监控点                       |

#### 查看程序变量和内存

| Command                   | Description                        |
| ------------------------- | ---------------------------------- |
| [args](#args)             | 打印函数参数                       |
| [display](#display)       | 每次程序停止时，打印一个表达式的值 |
| [examinemem](#examinemem) | 检查内存                           |
| [locals](#locals)         | 打印本地变量                       |
| [print](#print)           | 计算表达式                         |
| [regs](#regs)             | 打印寄存器的值                     |
| [set](#set)               | 修改变量值                         |
| [vars](#vars)             | 打印包变量                         |
| [whatis](#whatis)         | 获取表达式类型                     |

#### 查看和切换goroutine和线程

| Command                   | Description                  |
| ------------------------- | ---------------------------- |
| [goroutine](#goroutine)   | 显示或切换当前的Goroutine    |
| [goroutines](#goroutines) | 列出程序所有goroutine        |
| [thread](#thread)         | 切换到指定线程               |
| [threads](#threads)       | 打印出每个被追踪线程的信息。 |

#### 查看调用堆栈和选择栈帧

| Command               | Description                        |
| --------------------- | ---------------------------------- |
| [deferred](#deferred) | 在一个延迟调用的上下文中执行命令。 |
| [down](#down)         | 将当前帧向下移动                   |
| [frame](#frame)       | 设置当前帧，或在不同的帧上执行命令 |
| [stack](#stack)       | 打印堆栈记录                       |
| [up](#up)             | 将当前帧向上移动                   |

#### 其他命令

| Command                               | Description                                    |
| ------------------------------------- | ---------------------------------------------- |
| [check](#check)                       | 在当前位置创建一个检查点                       |
| [checkpoints](#checkpoints)           | 打印出现有检查点的信息                         |
| [clear-checkpoint](#clear-checkpoint) | 删除检查点                                     |
| [config](#config)                     | 改变配置参数                                   |
| [disassemble](#disassemble)           | 反汇编                                         |
| [dump](#dump)                         | 从当前进程状态创建一个coredump                 |
| [edit](#edit)                         | Open where you are in $DELVE_EDITOR or $EDITOR |
| [exit](#exit)                         | 退出                                           |
| [funcs](#funcs)                       | 打印函数列表                                   |
| [help](#help)                         | 打印帮助信息                                   |
| [libraries](#libraries)               | 列出加载的动态库                               |
| [list](#list)                         | 显示源码                                       |
| [source](#source)                     | 执行一个包含dlv命令列表的文件                  |
| [sources](#sources)                   | 打印源文件列表                                 |
| [types](#types)                       | 打印类型列表                                   |

#### args

打印函数参数。

	[goroutine <n>] [frame <m>] args [-v] [<regex>]

如果指定了regex，只有名称与之匹配的函数参数才会被返回。如果指定了-v，将显示每个函数参数的更多信息。

#### break

设置断点。

	break [name] <linespec>

参见[Documentation/cli/locspec.md](//github.com/go-delve/delve/tree/master/Documentation/cli/locspec.md)了解linespec的语法。

也请参见：”help on“、“help cond”和“help clear”。

别名: b

#### breakpoints

打印出生效断点的信息。

别名: bp

#### call

恢复进程，注入一个函数调用（实验性的！！！）

```
call [-unsafe] <function call expression>
```

当前限制:
- 只有指向栈分配的对象的指针可以作为参数传递。
- 只支持一些自动类型转换。
- functions can only be called on running goroutines that are not
  executing the runtime.
- 当前的goroutine栈上需要有至少256字节的空闲空间。
- functions can only be called when the goroutine is stopped at a safe
  point.
- 调用一个函数将恢复所有goroutine的执行。
- 只支持Linux的本地后端.

#### check

在当前位置创建一个检查点。

	checkpoint [note]

”note“是任意的文本，可用于标识检查点，如果没有指明，则默认为”filename:line position”。

别名: checkpoint

#### checkpoints

打印出现有检查点的信息。

#### clear

删除断点。

	clear <breakpoint name or id>

#### clear-checkpoint

删除检查点。

	clear-checkpoint <id>

別名: clearcheck

#### clearall

删除多个断点。

	clearall [<linespec>]

If called with the linespec argument it will delete all the breakpoints matching the linespec. If linespec is omitted all breakpoints are deleted.

如果调用指定linespec参数，它将删除所有与linespec匹配的断点。如果省略linespec，则所有断点都会被删除。

#### condition

设置断点条件。

	condition <breakpoint name or id> <boolean expression>.
	condition -hitcount <breakpoint name or id> <operator> <argument>

指定断点、跟踪点或观察点只有在布尔表达式为真时才会命中。

通过-hitcount选项，可以设置断点条件的命中数，支持以下运算符：

	condition -hitcount bp > n
	condition -hitcount bp >= n
	condition -hitcount bp < n
	condition -hitcount bp <= n
	condition -hitcount bp == n
	condition -hitcount bp != n
	condition -hitcount bp % n

“% n”形式意味着当hitcount是n的倍数时，我们应该在断点处停止。

别名: cond

#### config

改变配置参数。

	config -list

显示所有配置参数。

	config -save

将配置文件保存到磁盘，覆盖当前配置文件。

	config <parameter> <value>

改变配置参数值。

	config substitute-path <from> <to>
	config substitute-path <from>

添加或删除一个路径替换规则。

	config alias <command> <alias>
	config alias <alias>

将<alias>定义为<command>的别名或删除一个别名。

#### continue

运行至断点或程序终止。

	continue [<linespec>]

Optional linespec argument allows you to continue until a specific location is reached. The program will halt if a breakpoint is hit before reaching the specified location.

可选的linespec参数允许你继续执行直到到达一个特定的位置。如果在到达指定位置之前断点命中，程序将停止运行。

例如:

	continue main.main
	continue encoding/json.Marshal


别名: c

#### deferred

在一个延迟调用的上下文中执行命令。

	deferred <n> <command>

在当前帧中第n个延迟调用的上下文中执行指定的命令（print, args, locals）。

#### disassemble

反汇编。

	[goroutine <n>] [frame <m>] disassemble [-a <start> <end>] [-l <locspec>]

如果没有指定参数，在选定的栈帧中正在执行的函数将被反汇编。

	-a <start> <end>	反汇编指定的地址范围
	-l <locspec>		反汇编指定的函数

别名: disass

#### display

每次程序停止时，打印表达式的值。

	display -a [%format] <expression>
	display -d <number>

选项'-a'将一个表达式添加到每次程序停止时打印的表达式列表中。选项'-d'从列表中删除指定的表达式。

如果在没有参数的情况下调用display，它将打印列表中所有表达式的值。

#### down

将当前帧向下移动。

	down [<m>]
	down [<m>] <command>

将当前帧向下移动<m>。第二种形式是在给定的帧上运行该命令。

#### dump

从当前进程状态创建一个核心转储。

	dump <output file>

核心转储总是用ELF写的，即使是在不习惯这样做的系统上（windows, macOS）。对于linux/amd64以外的环境，线程和寄存器的转储格式只有Delve可以读回。

#### edit

Open where you are in $DELVE_EDITOR or $EDITOR

	edit [locspec]

如果省略locspec，edit将在编辑器中打开当前的源文件，否则它将打开指定的位置。

别名: ed

#### examinemem

查看内存：

	examinemem [-fmt <format>] [-count|-len <count>] [-size <size>] <address>
	examinemem [-fmt <format>] [-count|-len <count>] [-size <size>] -x <expression>

Format代表数据格式，其值是这个列表中的一个（默认为十六进制）：bin（二进制），oct（八进制），dec（十进制），hex（十六进制），addr（地址）。

长度是字节数（默认为1），必须小于或等于1000。

地址是要检查的目标的内存位置。请注意“-len已经被“-count”和“-size”所淘汰。

表达式可以是一个整数表达式或要检查的内存位置的指针值。

比如说。

    x -fmt hex -count 20 -size 1 0xc00008af38
    x -fmt hex -count 20 -size 1 -x 0xc00008af38 + 8
    x -fmt hex -count 20 -size 1 -x &myVar
    x -fmt hex -count 20 -size 1 -x myPtrVar

别名: x

#### exit

退出调试。

```
exit [-c]
```

当连接到一个用--accept-multiclient启动的`headless`实例时，在断开连接前通过-c来恢复目标进程的执行。

别名：quit q

#### frame

设置当前帧，或在不同的帧上执行命令。

	frame <m>					# 设置帧
	frame <m> <command>			# 在给定帧上执行命令

#### funcs

打印函数列表

	funcs [<regex>]

如果指定了regex，只有与之匹配的函数才会被返回。

#### goroutine

显示或改变当前的Goroutine。

	goroutine
	goroutine <id>
	goroutine <id> <command>

在没有参数的情况下调用，它将显示关于当前goroutine的信息。
用一个参数调用，它将切换到指定的goroutine。
用更多的参数调用，它将在指定的goroutine上执行一个命令。

别名：gr

#### goroutines

列出程序goroutines。

	goroutines [-u (default: user location)|-r (runtime location)|-g (go statement location)|-s (start location)] [-t (stack trace)] [-l (labels)]

打印出每个goroutine的信息。该标志控制了每个goroutine所显示的信息：

	-u	显示用户代码中最顶层栈帧的位置
	-r	显示最顶层的栈帧的位置 (包括私有运行时函数内部的帧)
	-g	显示创建goroutine的go指令的位置
	-s	显示函数开始的位置
	-t	显示goroutine的堆栈跟踪记录
	-l	显示goroutine的标签

如果没有指定标志，默认为-u。

别名：grs

#### help

打印帮助信息。

	help [command]

键入 "help"，并加上命令名，以获得更多相关信息。

#### libraries

列出已加载的动态库。

#### list

查看源码。

	[goroutine <n>] [frame <m>] list [<linespec>]

显示当前执行点或提供的linespec周围的源码。

比如:

	frame 1 list 69
	list testvariables.go:10000
	list main.main:30
	list 40

别名: ls l

#### locals

打印局部变量。

	[goroutine <n>] [frame <m>] locals [-v] [<regex>]

The name of variables that are shadowed in the current scope will be shown in parenthesis.

如果指定了regex，只有名称与之匹配的局部变量才会被返回。如果指定了-v，将显示每个局部变量的更多信息。

#### next

走到下一个代码行。

	next [count]

可选的[count]参数允许跳过多行。


别名：n

#### on

当断点命中，执行一个命令。

	on <breakpoint name or id> <command>.

支持的命令：（print、stack and goroutine）

#### print

打印表达式值

	[goroutine <n>] [frame <m>] print [%format] <expression>

关于支持的表达式的描述，请参见[Documentation/cli/expr.md](https://github.com/go-delve/delve/tree/master/Documentation/cli/expr.md)。

可选的`format`参数是一个格式指定符，就像`fmt`包所使用的那样。例如，"print %x v "将把v打印成一个十六进制的数字。

别名：p

#### rebuild

重建目标可执行文件并重新启动它。如果该可执行文件不是由delve构建的，它就不起作用。

#### regs

打印CPU寄存器的内容。

	regs [-a]

参数-a显示更多的寄存器。单个寄存器也可以通过“print”和“display”来显示。参见[Documentation/cli/expr.md.](//github.com/go-delve/delve/tree/master/Documentation/cli/expr.md.)

#### restart

重启进程。

For recorded targets the command takes the following forms:

	restart					resets to the start of the recording
	restart [checkpoint]			resets the recording to the given checkpoint
	restart -r [newargv...]	[redirects...]	re-records the target process

For live targets the command takes the following forms:

	restart [newargv...] [redirects...]	restarts the process

如果省略newargv，进程将以相同的参数向量重新启动（或重新记录）。
如果指定了-noargs，则参数向量被清除。

可以在新的参数列表后指定一个文件重定向列表，以覆盖使用"--重定向 "命令行选项定义的重定向。使用的是类似于Unix shells的语法。    

	<input.txt	redirects the standard input of the target process from input.txt
	>output.txt	redirects the standard output of the target process to output.txt
	2>error.txt	redirects the standard error of the target process to error.txt


别名：r

#### set

改变变量的值。

	[goroutine <n>] [frame <m>] set <variable> = <value>

关于支持的表达式的描述，请参见[Documentation/cli/expr.md](https://github.com/go-delve/delve/tree/master/Documentation/cli/expr.md)。只有数字型变量和指针可以被改变。

#### source

执行一个包含delve命令列表的文件。

	source <path>

如果`path`以.star扩展名结尾，它将被解释为一个starlark脚本。语法见 [Documentation/cli/starlark.md](//github.com/go-delve/telve/master/Documentation/cli/starlark.md)。

如果`path`是一个单一的’-‘字符，一个交互式的starlark解释器将被启动。键入”exit“来退出。

#### sources

打印源文件的清单。

	sources [<regex>]

如果指定了regex，只有与之匹配的源文件才会被返回。

#### stack

打印堆栈跟踪。

	[goroutine <n>] [frame <m>] stack [<depth>] [-full] [-offsets] [-defer] [-a <n>] [-adepth <depth>] [-mode <mode>]
	
	-full		every stackframe is decorated with the value of its local variables and arguments.
	-offsets	prints frame offset of each frame.
	-defer		prints deferred function call stack for each frame.
	-a <n>		prints stacktrace of n ancestors of the selected goroutine (target process must have tracebackancestors enabled)
	-adepth <depth>	configures depth of ancestor stacktrace
	-mode <mode>	specifies the stacktrace mode, possible values are:
			normal	- attempts to automatically switch between cgo frames and go frames
			simple	- disables automatic switch between cgo and go
			fromg	- starts from the registers stored in the runtime.g struct


别名：bt

#### step

单步执行程序

别名：s

#### step-instruction

单步执行cpu指令

别名：si

#### stepout

走出当前函数

别名：so

#### thread

切换到指定的线程。

	thread <id>

别名：tr

#### threads

打印出每个被追踪线程的信息。

#### toggle

打开或关闭一个断点。

```
toggle <breakpoint name or id>
```

#### trace

设置跟踪点。

	trace [name] <linespec>

跟踪点是一个不会停止程序执行的断点，相反，当跟踪点被击中时，会显示一个通知。请参阅[Documentation/cli/locspec.md](//github.com/go-delve/delve/tree/master/Documentation/cli/locspec.md)了解linespec的语法。

也请参见: "help on", "help cond" and "help clear"

别名：t

#### types

打印类型列表。

	types [<regex>]

如果指定了regex，只有与之匹配的类型才会被返回。

#### up

将当前帧向上移动。

	up [<m>]				# 将当前帧向上移动<m>
	up [<m>] <command>		# 在给定的帧上运行该命令

#### vars

打印包变量。

	vars [-v] [<regex>]

如果指定了regex，只有名称与之匹配的包变量才会被返回。如果指定了-v，将显示关于每个包变量的更多信息。

#### watch

设置观察点。

```
watch [-r|-w|-rw] <expr>

-r	当内存位置被读取时停止
-w	当内存位置被写入时停止
-rw	当内存位置被读取或写入时停止
```

内存位置是与'print'所使用的表达式语言一样来指定的，例如：

	watch v

将观察变量”v“的地址。

参考: "help print".

#### whatis

打印表达式的类型。

	whatis <expression>

### 常见问题

#### 如何在docker中使用dlv

启动docker容器时，增加参数`--security-opt=seccomp:unconfined`。

你可以像这样在容器内启动一个Delve的`headless`实例:

```bash
$ dlv exec --headless --listen :4040 /path/to/executable
```

然后从容器外连接到它:

```bash
$ dlv connect :4040
```

程序不会开始执行，直到你连接到Delve并发送`continue`命令。 如果你想让程序立即开始，你可以通过向Delve传递`--continue`和`--accept-multiclient`选项来实现:

```bash
$ dlv exec --headless --continue --listen :4040 --accept-multiclient /path/to/executable
```

注意，与Delve的连接是未经认证的，将允许任意的远程代码执行。*请不要在生产中这样做*。

#### 如何使用Delve来调试CLI应用程序

1. 在一个单独的终端中运行你的CLI应用程序，然后通过`dlv attach`调试。
1. 通过`dlv debug --headless`在`headless`模式下运行Delve，然后从另一个终端连接到它。这将使该进程处于前台，并允许它访问终端TTY。
1. 为进程指定它自己的TTY。这在UNIX系统上可以使用`dlv debug`和`dlv exec`命令通过`--tty`标志来实现。为了获得最好的体验，你应该创建你自己的PTY并将其指定为TTY。这可以通过[ptyme](https://github.com/derekparker/ptyme)完成。

#### 如何使用dlv进行远程调试

建议最好不要在公网上使用远程调试。如果你必须远程调试，我们建议使用ssh隧道或者VPN连接。

**示例** 

远程服务端:

```bash
$ dlv exec --headless --listen localhost:4040 /path/to/executable
```

本地客户端:
1. 连接到服务端并且启动一个本地端口转发

```bash
$ ssh -NL 4040:localhost:4040 user@remote.ip
```

2. 连接本地端口
```bash
$ dlv connect :4040
```

### 参考文章

[delve usage](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv.md)

[Command line client](https://github.com/go-delve/delve/blob/master/Documentation/cli/README.md)

[Getting Started](https://github.com/go-delve/delve/blob/master/Documentation/cli/getting_started.md)

[delve FAQ](https://github.com/go-delve/delve/blob/master/Documentation/faq.md)

