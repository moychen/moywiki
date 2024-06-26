# Go语言是如何启动的

## 理解可执行文件

### 程序构建过程

```bash
$ go build -x go文件	# -x 打印构建详细过程
```

```txt
$ go build -x demo.go
WORK=/tmp/go-build650300936
mkdir -p $WORK/b001/
cat >$WORK/b001/importcfg << 'EOF' # internal
# import config
packagefile fmt=/usr/local/go/pkg/linux_amd64/fmt.a
packagefile runtime=/usr/local/go/pkg/linux_amd64/runtime.a
EOF
cd /root/moychen/demo
-------------------------------编译-----------------------------------
/usr/local/go/pkg/tool/linux_amd64/compile -o $WORK/b001/_pkg_.a -trimpath "$WORK/b001=>" -p main -complete -buildid IHvcy5J3sPf6q7qSX-yU/IHvcy5J3sPf6q7qSX-yU -goversion go1.15.2 -D _/root/moychen/demo -importcfg $WORK/b001/importcfg -pack ./demo.go
/usr/local/go/pkg/tool/linux_amd64/buildid -w $WORK/b001/_pkg_.a # internal
cp $WORK/b001/_pkg_.a /root/.cache/go-build/5d/5d735f650ab1e82b957e556236d1f63ac6f76a8dcd5e347a1f7c98d556a8b37f-d # internal
cat >$WORK/b001/importcfg.link << 'EOF' # internal
packagefile command-line-arguments=$WORK/b001/_pkg_.a
...
packagefile internal/race=/usr/local/go/pkg/linux_amd64/internal/race.a
EOF
mkdir -p $WORK/b001/exe/
cd .
-------------------------------- 链接----------------------------------
/usr/local/go/pkg/tool/linux_amd64/link -o $WORK/b001/exe/a.out -importcfg $WORK/b001/importcfg.link -buildmode=exe -buildid=VljoluUulGgW80qhU4WS/IHvcy5J3sPf6q7qSX-yU/CUJ3YEw9E6g_F0Y_-2iM/VljoluUulGgW80qhU4WS -extld=gcc $WORK/b001/_pkg_.a
/usr/local/go/pkg/tool/linux_amd64/buildid -w $WORK/b001/exe/a.out # internal
mv $WORK/b001/exe/a.out demo
rm -r $WORK/b001/
```

### 可执行文件

可执行文件在不同平台格式规范也不一致。以Linux下ELF（Executable and Linkable Format）为例，详细内容参考[ELF规范参考文档](https://github.com/corkami/pics/blob/28cb0226093ed57b348723bc473cea0162dad366/binary/elf101/elf101.pdf[)，ELF文件主要组成部分有：

> * ELF header
> * Section header
> * Sections

![image-20210519180400990](images/Go程序是如何启动的/image-20210519180400990.png)

如上图所示，通过ELF header中的 entry point 能够找到 Go 进程的执行入口。Linux下运行可执行文件的步骤大致如下：

```mermaid
graph LR;
解析ELF-Header --> 加载二进制文件内容至内存 --> 从entry-point开始执行代码;
```

## Go进程的启动和初始化

生成二进制可执行文件后，启动时计算机通过执行二进制机器码指令运行程序，每执行一条语句pc寄存器就指向下一条指令。在 64 位平台上 pc 寄存器 = rip。

```go
// demo.go 
package main

import (
	"fmt"
)

func main() {
	fmt.Println("hello world")
}
```

go build 生成可执行文件demo，然后通过readelf -h demo找到可执行文件demo的entry point找到程序入口，使用dlv调试程序。

![image-20210520210757578](images/Go程序是如何启动的/image-20210520210757578.png)

中间省略一下没有用的信息。

![image-20210520211303257](images/Go程序是如何启动的/image-20210520211303257.png)

从上图的结果来看go启动流程大致为：

```mermaid
graph TD;
_rt0_amd64_linux --> _rt0_amd64 --> runtime.rt0_go; 
```

runtime.rt0_go中主要工作有：

```mermaid
graph LR;
argc/argv处理 --> 全局m0/g0初始化 --> 获取cpu核心数 --> 初始化内部数据结构 --> 开始执行main函数;
```

## 调度组件与调度循环

**G（goroutine）**：⼀个计算任务。由需要执⾏的代码和其上下⽂组成，上下⽂包括：当前代码位置、栈顶、栈底地址、状态等。

**M（machine）**：系统线程，执⾏实体，想要在 CPU 上执⾏代码，必须有线程，与 C 语⾔中的线程相同，通过系统调⽤ clone 来创建。

**P（processor）**：虚拟处理器，M 必须获得 P 才能执行代码，否则必须陷⼊休眠（后台监控线程除外），你也可以将其理解为⼀种 token，有这个 token，才有在物理 CPU 核心上执行的权利。

### 调度流程

Go的调度流程实际上是一个生产-消费流程，当我们新增一个go routine的时候实际上是向runtime 调度组件中新增一个task。

![image-20210521163354177](images/Go程序是如何启动的/image-20210521163354177.png)

### 生产者

<img src="images/Go程序是如何启动的/image-20210521165029181.png" alt="image-20210521165029181" style="zoom:67%;" />

### 消费者

![image-20210521165214744](images/Go程序是如何启动的/image-20210521165214744.png)

> * M 执⾏调度循环时，必须与⼀个 P 绑定
> * Work stealing 就是说的 runqsteal -> runqgrab 这个流程

<img src="images/Go程序是如何启动的/image-20210521165441266.png" alt="image-20210521165441266" style="zoom: 50%;" />

## 处理阻塞

Go中阻塞大概有这么几种场景：

> * channel send/recv
> * sleep
> * select
> * lock
> * 网络IO

当上述阻塞场景之一发生时，Go会将该goroutine挂起，不会阻塞Go的调度流程。挂起指的是调度组件会将该goroutine放入等待队列，待可执行时，待ready后再继续执行，不会占用线程。

### 可拦截的阻塞

#### channel send/recv

#### time.Sleep

#### select

#### lock

#### 网络IO

### 不可拦截的阻塞

> * cgo
> * syscall 系统调用
> * ⻓时间运⾏需要剥离 P 执⾏

### sysmon

sysmon是runtime中的system monitor，具有高优先级，在专有线程中执行，并且不需要绑定P。

checkdead：常见误解—这个可以检查死锁

netpoll：inject g list to global run queue

retake：如果syscall卡了很久，就把剥离（handoff p）

​				如果用户g运行很久，那么发信号SIGURG抢占

## 与调度有关的常见问题

