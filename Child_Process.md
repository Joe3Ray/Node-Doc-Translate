##子进程
===

	稳定性：3 - 稳定的
	
Node通过`child_process`模块提供了类似`popen(3)`的处理三向流的功能。

它能够以完全非阻塞的方式与子进程的`stdin`，`stdout`和`stderr`以流来传递数据。（注意，某些程序在内部使用行缓冲I/O。这不会影响node.js，不过这意味着你传递给子进程的数据可能不会被立即消费。）

使用`require('child_process').spawn()`或者`require('child_process').fork()`来创建一个子进程。两者的语义略有不同，后面会解释。

你可能会发现同步的方式编程会更方便。

###类：ChildProcess
`ChildProcess`是一个EventEmitter。