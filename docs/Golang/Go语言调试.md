Go语言调试

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
| [rev](#rev)                           | Reverses the execution of the target program for the command specified. |
| [rewind](#rewind)                     | 向后运行直到断点或程序终止                                   |
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

| Command               | Description                                         |
| --------------------- | --------------------------------------------------- |
| [deferred](#deferred) | Executes command in the context of a deferred call. |
| [down](#down)         | 将当前帧向下移动                                    |
| [frame](#frame)       | 设置当前帧，或在不同的帧上执行命令                  |
| [stack](#stack)       | 打印堆栈记录                                        |
| [up](#up)             | 将当前帧向上移动                                    |

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
	call [-unsafe] <function call expression>

当前限制:
- only pointers to stack-allocated objects can be passed as argument.
- only some automatic type conversions are supported.
- functions can only be called on running goroutines that are not
  executing the runtime.
- the current goroutine needs to have at least 256 bytes of free space on
  the stack.
- functions can only be called when the goroutine is stopped at a safe
  point.
- calling a function will resume execution of all goroutines.
- only supported on linux's native backend.

#### check

Creates a checkpoint at the current position.

	checkpoint [note]

The "note" is arbitrary text that can be used to identify the checkpoint, if it is not specified it defaults to the current filename:line position.

Aliases: checkpoint

#### checkpoints

Print out info for existing checkpoints.

#### clear

Deletes breakpoint.

	clear <breakpoint name or id>

#### clear-checkpoint

Deletes checkpoint.

	clear-checkpoint <id>

Aliases: clearcheck

#### clearall

Deletes multiple breakpoints.

	clearall [<linespec>]

If called with the linespec argument it will delete all the breakpoints matching the linespec. If linespec is omitted all breakpoints are deleted.

#### condition

Set breakpoint condition.

	condition <breakpoint name or id> <boolean expression>.
	condition -hitcount <breakpoint name or id> <operator> <argument>

Specifies that the breakpoint, tracepoint or watchpoint should break only if the boolean expression is true.

With the -hitcount option a condition on the breakpoint hit count can be set, the following operators are supported

	condition -hitcount bp > n
	condition -hitcount bp >= n
	condition -hitcount bp < n
	condition -hitcount bp <= n
	condition -hitcount bp == n
	condition -hitcount bp != n
	condition -hitcount bp % n

The '% n' form means we should stop at the breakpoint when the hitcount is a multiple of n.

Aliases: cond

#### config

Changes configuration parameters.

	config -list

Show all configuration parameters.

	config -save

Saves the configuration file to disk, overwriting the current configuration file.

	config <parameter> <value>

Changes the value of a configuration parameter.

	config substitute-path <from> <to>
	config substitute-path <from>

Adds or removes a path substitution rule.

	config alias <command> <alias>
	config alias <alias>

Defines <alias> as an alias to <command> or removes an alias.

#### continue

Run until breakpoint or program termination.

	continue [<linespec>]

Optional linespec argument allows you to continue until a specific location is reached. The program will halt if a breakpoint is hit before reaching the specified location.

For example:

	continue main.main
	continue encoding/json.Marshal


Aliases: c

#### deferred

Executes command in the context of a deferred call.

	deferred <n> <command>

Executes the specified command (print, args, locals) in the context of the n-th deferred call in the current frame.

#### disassemble

Disassembler.

	[goroutine <n>] [frame <m>] disassemble [-a <start> <end>] [-l <locspec>]

If no argument is specified the function being executed in the selected stack frame will be executed.

	-a <start> <end>	disassembles the specified address range
	-l <locspec>		disassembles the specified function

Aliases: disass

#### display

Print value of an expression every time the program stops.

	display -a [%format] <expression>
	display -d <number>

The '-a' option adds an expression to the list of expression printed every time the program stops. The '-d' option removes the specified expression from the list.

If display is called without arguments it will print the value of all expression in the list.

#### down

Move the current frame down.

	down [<m>]
	down [<m>] <command>

Move the current frame down by <m>. The second form runs the command on the given frame.

#### dump

Creates a core dump from the current process state

	dump <output file>

The core dump is always written in ELF, even on systems (windows, macOS) where this is not customary. For environments other than linux/amd64 threads and registers are dumped in a format that only Delve can read back.

#### edit

Open where you are in $DELVE_EDITOR or $EDITOR

	edit [locspec]

If locspec is omitted edit will open the current source file in the editor, otherwise it will open the specified location.

Aliases: ed

#### examinemem

Examine memory:

	examinemem [-fmt <format>] [-count|-len <count>] [-size <size>] <address>
	examinemem [-fmt <format>] [-count|-len <count>] [-size <size>] -x <expression>

Format represents the data format and the value is one of this list (default hex): bin(binary), oct(octal), dec(decimal), hex(hexadecimal), addr(address).
Length is the number of bytes (default 1) and must be less than or equal to 1000.
Address is the memory location of the target to examine. Please note '-len' is deprecated by '-count and -size'.
Expression can be an integer expression or pointer value of the memory location to examine.

For example:

    x -fmt hex -count 20 -size 1 0xc00008af38
    x -fmt hex -count 20 -size 1 -x 0xc00008af38 + 8
    x -fmt hex -count 20 -size 1 -x &myVar
    x -fmt hex -count 20 -size 1 -x myPtrVar

Aliases: x

#### exit

Exit the debugger.	
	exit [-c]

When connected to a headless instance started with the --accept-multiclient, pass -c to resume the execution of the target process before disconnecting.

Aliases: quit q

#### frame

Set the current frame, or execute command on a different frame.

	frame <m>
	frame <m> <command>

The first form sets frame used by subsequent commands such as "print" or "set".
The second form runs the command on the given frame.

#### funcs

Print list of functions.

	funcs [<regex>]

If regex is specified only the functions matching it will be returned.

#### goroutine

Shows or changes current goroutine

	goroutine
	goroutine <id>
	goroutine <id> <command>

Called without arguments it will show information about the current goroutine.
Called with a single argument it will switch to the specified goroutine.
Called with more arguments it will execute a command on the specified goroutine.

Aliases: gr

#### goroutines

List program goroutines.

	goroutines [-u (default: user location)|-r (runtime location)|-g (go statement location)|-s (start location)] [-t (stack trace)] [-l (labels)]

Print out info for every goroutine. The flag controls what information is shown along with each goroutine:

	-u	displays location of topmost stackframe in user code
	-r	displays location of topmost stackframe (including frames inside private runtime functions)
	-g	displays location of go instruction that created the goroutine
	-s	displays location of the start function
	-t	displays goroutine's stacktrace
	-l	displays goroutine's labels

If no flag is specified the default is -u.

Aliases: grs

#### help

Prints the help message.

	help [command]

Type "help" followed by the name of a command for more information about it.

Aliases: h

#### libraries

List loaded dynamic libraries

#### list

Show source code.

	[goroutine <n>] [frame <m>] list [<linespec>]

Show source around current point or provided linespec.

For example:

	frame 1 list 69
	list testvariables.go:10000
	list main.main:30
	list 40

Aliases: ls l

#### locals

Print local variables.

	[goroutine <n>] [frame <m>] locals [-v] [<regex>]

The name of variables that are shadowed in the current scope will be shown in parenthesis.

If regex is specified only local variables with a name matching it will be returned. If -v is specified more information about each local variable will be shown.

#### next

Step over to next source line.

	next [count]

Optional [count] argument allows you to skip multiple lines.


Aliases: n

#### on

Executes a command when a breakpoint is hit.

	on <breakpoint name or id> <command>.

Supported commands: print, stack and goroutine)

#### print

Evaluate an expression.

	[goroutine <n>] [frame <m>] print [%format] <expression>

See [Documentation/cli/expr.md](//github.com/go-delve/delve/tree/master/Documentation/cli/expr.md) for a description of supported expressions.

The optional format argument is a format specifier, like the ones used by the fmt package. For example "print %x v" will print v as an hexadecimal number.

Aliases: p

#### rebuild

Rebuild the target executable and restarts it. It does not work if the executable was not built by delve.

#### regs

Print contents of CPU registers.

	regs [-a]

Argument -a shows more registers. Individual registers can also be displayed by 'print' and 'display'. See [Documentation/cli/expr.md.](//github.com/go-delve/delve/tree/master/Documentation/cli/expr.md.)

#### restart

Restart process.

For recorded targets the command takes the following forms:

	restart					resets to the start of the recording
	restart [checkpoint]			resets the recording to the given checkpoint
	restart -r [newargv...]	[redirects...]	re-records the target process

For live targets the command takes the following forms:

	restart [newargv...] [redirects...]	restarts the process

If newargv is omitted the process is restarted (or re-recorded) with the same argument vector.
If -noargs is specified instead, the argument vector is cleared.

A list of file redirections can be specified after the new argument list to override the redirections defined using the '--redirect' command line option. A syntax similar to Unix shells is used:

	<input.txt	redirects the standard input of the target process from input.txt
	>output.txt	redirects the standard output of the target process to output.txt
	2>error.txt	redirects the standard error of the target process to error.txt


Aliases: r

#### rev

Reverses the execution of the target program for the command specified.
Currently, only the rev step-instruction command is supported.

#### rewind

Run backwards until breakpoint or program termination.

Aliases: rw

#### set

Changes the value of a variable.

	[goroutine <n>] [frame <m>] set <variable> = <value>

See [Documentation/cli/expr.md](//github.com/go-delve/delve/tree/master/Documentation/cli/expr.md) for a description of supported expressions. Only numerical variables and pointers can be changed.

#### source

Executes a file containing a list of delve commands

	source <path>

If path ends with the .star extension it will be interpreted as a starlark script. See [Documentation/cli/starlark.md](//github.com/go-delve/delve/tree/master/Documentation/cli/starlark.md) for the syntax.

If path is a single '-' character an interactive starlark interpreter will start instead. Type 'exit' to exit.

#### sources

Print list of source files.

	sources [<regex>]

If regex is specified only the source files matching it will be returned.

#### stack

Print stack trace.

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


Aliases: bt

#### step

Single step through program.

Aliases: s

#### step-instruction

Single step a single cpu instruction.

Aliases: si

#### stepout

Step out of the current function.

Aliases: so

#### thread

Switch to the specified thread.

	thread <id>

Aliases: tr

#### threads

Print out info for every traced thread.

#### toggle

Toggles on or off a breakpoint.

```
toggle <breakpoint name or id>
```

#### trace

Set tracepoint.

	trace [name] <linespec>

A tracepoint is a breakpoint that does not stop the execution of the program, instead when the tracepoint is hit a notification is displayed. See [Documentation/cli/locspec.md](//github.com/go-delve/delve/tree/master/Documentation/cli/locspec.md) for the syntax of linespec.

See also: "help on", "help cond" and "help clear"

Aliases: t

#### types

Print list of types

	types [<regex>]

If regex is specified only the types matching it will be returned.

#### up

Move the current frame up.

	up [<m>]
	up [<m>] <command>

Move the current frame up by <m>. The second form runs the command on the given frame.

#### vars

Print package variables.

	vars [-v] [<regex>]

If regex is specified only package variables with a name matching it will be returned. If -v is specified more information about each package variable will be shown.

#### watch

Set watchpoint.
	watch [-r|-w|-rw] <expr>
	
	-r	stops when the memory location is read
	-w	stops when the memory location is written
	-rw	stops when the memory location is read or written

The memory location is specified with the same expression language used by 'print', for example:

	watch v

will watch the address of variable 'v'.

See also: "help print".

#### whatis

Prints type of an expression.

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

