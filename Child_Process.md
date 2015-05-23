##子进程
===

	稳定性：3 - 稳定的
	
Node通过`child_process`模块提供了类似`popen(3)`的处理三向流的功能。

它能够以完全非阻塞的方式与子进程的`stdin`，`stdout`和`stderr`以流来传递数据。（注意，某些程序在内部使用行缓冲I/O。这不会影响node.js，不过这意味着你传递给子进程的数据可能不会被立即消费。）

使用`require('child_process').spawn()`或者`require('child_process').fork()`来创建一个子进程。两者的语义略有不同，后面会解释。

你可能会发现同步的方式编程会更方便。

###类：ChildProcess
`ChildProcess`是一个EventEmitter。

子进程始终有3个与它们相关的数据流。`child.stdin`，`child.stdout`和`child.stderr`。它们可以共享父进程的stdio流，也可以作为独立的被导流的流对象。

ChildProcess类不能被直接使用。使用`spawn()`、`exec()`、`execFile()`或者`fork()`方法来创建一个子进程实例。

####事件：'error'
* `err` 错误对象 错误

触发于：

1. 进程不能被创建，或者
2. 进程不能被终止，或者
3. 由任何原因引起的发送消息到子进程失败

注意，错误出现后`exit`事件可能会也可能不会被触发。如果你同时监听了这2个事件来触发一个函数，记住要避免函数被调用2次。

请参考` ChildProcess#kill()`和` ChildProcess#send()`。

####事件：'exit'
* `code` 数字 如果正常退出，这是退出代码
* `signal` 字符串 如果是被父进程终止，这个就是传过来终止子进程的信号

这个事件在子进程终止后触发。如果进程正常终止，`code`就是进程最终的退出代码，否则为`null`。如果进程是接收到一个信号而终止，`signal`就是信号的字符串名字，否则为`null`。

注意子进程的stdio流可能仍为开启状态。

同样要注意node为`'SIGINT'`和`'SIGTERM'`信号创建了处理方法，所以接收到这些信号时它不会终止，而是退出。

参考`waitpid(2)`。

####事件：'close'
* `code` 数字 如果正常退出，这是退出代码
* `signal` 字符串 如果是被父进程终止，这个就是传过来终止子进程的信号

这个事件在子进程的stdio流全部终止时触发。这和'exit'不同，因为多进程可能共享同一个stdio流。

####事件：'disconnect'

这个事件在父进程或者子进程调用`.disconnect()`方法时触发。在断开连接后就不可以发送消息，`.connected`属性也会变成false。

####事件：'message'
* `message` 对象 一个已经解析的JSON对象或者原始类型的值
* `sendHandle` Handle对象 一个socket或者server对象

使用`.send(message, [sendHandle])`方法发送的信息可以通过监听`message`事件获取。

####child.stdin
* 流对象

一个可写的流，代表子进程的`stdin`。如果子进程正在等待读取它的所有输出，那么它将在通过`end()`方法关闭这个流之后才能继续工作。

如果子进程不是通过设置为`'pipe'`的`stdio[0]`生成的，那么它将不会设置。

`child.stdin`是`child.stdio[0]`的简写。两个属性指向同一个对象，或是null。

####child.stdout
* 流对象

一个可读的流，代表子进程的`stdout`。

如果子进程不是通过设置为`'pipe'`的`stdio[1]`生成的，那么它将不会设置。

`child.stdout`是`child.stdio[1]`的简写。两个属性指向同一个对象，或是null。

####child.stderr
* 流对象

一个可读的流，代表子进程的`stderr`。

如果子进程不是通过设置为`'pipe'`的`stdio[2]`生成的，那么它将不会设置。

`child.stderr`是`child.stdio[2]`的简写。两个属性指向同一个对象，或是null。

####child.stdio
* 数组

一个简朴的通往子进程的管道的数组，对应设置为`'pipe'`的spwan的stdio选项的位置。注意流0-2也可分别作为ChildProcess.stdin、ChildProcess.stdout和ChildProcess.stderr。

在下面的例子中，只有子进程的索引为`1`的位置被设置为pipe，所以只有父进程的`child.stdio[1]`是流，数组中的其他值全都是`null`。

	child = child_process.spawn("ls", {
	    stdio: [
	      0, // use parents stdin for child
	      'pipe', // pipe child's stdout to parent
	      fs.openSync("err.out", "w") // direct child's stderr to a file
	    ]
	});
	
	assert.equal(child.stdio[0], null);
	assert.equal(child.stdio[0], child.stdin);
	
	assert(child.stdout);
	assert.equal(child.stdio[1], child.stdout);
	
	assert.equal(child.stdio[2], null);
	assert.equal(child.stdio[2], child.stderr);
	
