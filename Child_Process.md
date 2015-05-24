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
	
####child.pid
* 数字

子进程的PID。

例子：

	var spawn = require('child_process').spawn,
	    grep = spawn('grep', ['ssh']);
		
    console.log('Spawned child pid: ' + grep.pid);
	grep.stdin.end();
	
####child.connected
* 布尔值 在调用'.disconnect'后被设置成false

如果`.connected`是false，就不能再发送消息。

####child.kill([signal])
* `signal` 字符串

向子进程发送一个信号。如果没有提供任何参数，将会发送`'SIGTERM'`。查看`signal(7)`可以看到所有用信号的列表。

	var spawn = require('child_process').spawn,
	    grep = spawn('grep', ['ssh']);
		
	grep.on('close', function (code, signal) {
	    console.log('child process terminated due to receipt of signal '+signal);
	});
	
	// send SIGHUP to process
	grep.kill('SIGHUP');
	
当信号无法传递时可能会触发`'error'`事件。给一个已经退出的子进程发信号不会有错误但是可能会有没预料到的后果：如果PID(进程id)被分配给其他进程，信号将会发送给那个进程。接下来会发生什么就不好说了。

注意虽然这个方法被叫做`kill`，但是传递给子进程的信号可能不会真的终止它。`kill`实际上只是给一个进程发送信号。

参考`kill(2)`。

####child.send(message[, sendHandle])
* `message` 对象
* `sendHandle` Handle对象

当使用`child_process.fork()`时你可以通过`child.send(message[, sendHandle])`写入子进程，子进程通过监听`message`事件可以捕获这个消息。

例如：

	var cp = require('child_process');
	
	var n = cp.fork(__dirname + '/sub.js');
	
	n.on('message', function (m) {
	    console.log('PARENT got message:', m);
	});
	
	n.send({ hello: 'world' });
	
子进程的脚本`'sub.js'`可能是这样的：

	process.on('message', function (m) {
	    console.log('CHILD got message:', m);
	});
	
	process.send({foo: 'bar'});
	
子进程中`process`对象有一个`send()`方法，`process`每次通过它的信道接收到信息都会触发事件，信息是一个对象。

请注意，`send()`方法在父子进程中都是同步的 - 不建议发送大块的数据。（可以使用pipes来替代，查看`child_process.spawn`）。

发送`{md: 'NODE_foo'}`有一个特殊的情况。所有在`cmd`属性中包含一个`NODE_`前缀的消息都不会触发`message`事件，因为他们是node核心使用的内部消息。包含这个前缀的消息会触发`internalMessage`事件，你应该尽量避免使用这个特性，它有可能随时更改但不通知你。

`child.send()`的`sendHandle`选项是为了把一个TCP socket或者server对象发送给其他进程。子进程可以监听`message`事件，从第二个参数获取这个对象。

发送不了消息的时候会触发`error`事件，例如子进程已经退出的情况。

**示例：发送server对象#**

下面是一个发送server的示例：

	var child = require('child_process').fork('child.js');
	
	// Open up the server object and send the handle.
	var server = require('net').createServer();
	server.on('connection', function (socket) {
	  socket.end('handled by parent');
	});
	server.listen(1337, function() {
	  child.send('server', server);
	});
	
子进程将这样处理server对象：

	process.on('message', function(m, server) {
	  if (m === 'server') {
	    server.on('connection', function (socket) {
	      socket.end('handled by child');
	    });
	  }
	});
	
现在你需要注意server是父子进程共享的，这意味着有的连接是父进程处理，有的连接是子进程处理。

对`dgram`服务器来说，工作流程是一样的。你监听的是`message`事件，而不是 `connection`事件， 使用`server.bind` ,而不是`server.listen`。(当前仅在UNIX平台支持。)

**示例：发送socket对象**

下面是一个发送socket对象的示例。他将创建两个子线程 ，同时处理连接，这是通过使用远程地址`74.125.127.100`作为 VIP 发送socket到一个‘特殊’的子线程， 其他的socket将会发送到‘正常’的线程里。

	var normal = require('child_process').fork('child.js', ['normal']);
	var special = require('child_process').fork('child.js', ['special']);
	
	// Open up the server and send sockets to child
	var server = require('net').createServer();
	server.on('connection', function (socket) {
	
	  // if this is a VIP
	  if (socket.remoteAddress === '74.125.127.100') {
	    special.send('socket', socket);
	    return;
	  }
	  // just the usual dudes
	  normal.send('socket', socket);
	});
	server.listen(1337);
	
`child.js`的脚本代码应该是这样的：

	process.on('message', function(m, socket) {
	  if (m === 'socket') {
	    socket.end('You were handled as a ' + process.argv[2] + ' person');
	  }
	});
	
注意，一旦单个的socket被发送到子进程，当这个socket被删除之后，父进程将不再对它保存跟踪，这表明了这个条件下`.connetions`属性将变成'null'， 在这个条件下同时也不推荐使用`.maxConnections`属性。

####child.disconnect()

关闭父子进程之间的IPC通道，在没有其他连接让他保持活动状态的情况下，子进程将优雅地退出。调用这个函数后`.connected`属性在父子进程中都将被设置为`false`，之后就不能再发送消息。

'disconnect'事件在进程接收不到任何消息时将被触发，很可能是立即触发。

需要注意的是你同样可以在子进程中调用`process.disconnect()`，如果子进程和父进程之间有一个打开的IPC通道的话（比如`fork()`）。

###异步创建进程

这些方法遵循常见的异步编程模式（接受一个回调或者返回一个EventEmitter）。

####child_process.spawn(command[, args][, options])
* `command` 字符串 需要运行的命令
* `args` 数组 参数字符串列表
* `options` 对象
 * `cwd` 字符串 子进程当前的工作目录
 * `env` 对象 环境键值对
 * `stdio` 数组|字符串 子进程的stdio配置。（见下面）
 * `customFds` 数组 **废弃** 子进程为了stdio使用的文件描述符。（见下面）
 * `detached` 布尔值 子进程将是一个进程组组长。（见下面）
 * `uid` 数字 设置进程的用户身份
 * `gid` 数字 设置进程的用户组身份
* 返回：子进程对象

根据给定的命令和和`args`中的命令行参数启动一个新的进程。若省略，`args`默认是一个空数组。

第三个参数作用是指定额外选项，有这些默认值：

	{ cwd: undefined,
	  env: process.env
	}
	
使用`cwd`来指定生成该进程的工作目录。如果没有给定，默认为当前进程的工作目录。

使用`env`来指定新进程中的环境变量，默认为process.env。

一个运行`ls -lh /usr`的例子，捕获`stdout`、`stderr`和退出代码：

	var spawn = require('child_process').spawn,
	    ls    = spawn('ls', ['-lh', '/usr']);
	
	ls.stdout.on('data', function (data) {
	  console.log('stdout: ' + data);
	});
	
	ls.stderr.on('data', function (data) {
	  console.log('stderr: ' + data);
	});
	
	ls.on('close', function (code) {
	  console.log('child process exited with code ' + code);
	});
	
例子：一个非常复杂的运行'ps ax | grep ssh'的方法

	var spawn = require('child_process').spawn,
	    ps    = spawn('ps', ['ax']),
	    grep  = spawn('grep', ['ssh']);
	
	ps.stdout.on('data', function (data) {
	  grep.stdin.write(data);
	});
	
	ps.stderr.on('data', function (data) {
	  console.log('ps stderr: ' + data);
	});
	
	ps.on('close', function (code) {
	  if (code !== 0) {
	    console.log('ps process exited with code ' + code);
	  }
	  grep.stdin.end();
	});
	
	grep.stdout.on('data', function (data) {
	  console.log('' + data);
	});
	
	grep.stderr.on('data', function (data) {
	  console.log('grep stderr: ' + data);
	});
	
	grep.on('close', function (code) {
	  if (code !== 0) {
	    console.log('grep process exited with code ' + code);
	  }
	});
	
####options.stdio

作为一个简写，`stdio`参数可能是下列字符串中的一个：
* `'pipe'` - `['pipe', 'pipe', 'pipe']`，这是默认值
* `'ignore'` - `['ignore', 'ignore', 'ignore']`
* `'inherit'` - `[process.stdin, process.stdout, process.stderr]`或者`[0, 1, 2]`

否则，`child_process.spawn()`的stdio选项就是一个数组，每一个位置都代表了子进程的一个fd。可用值是下列几个之一：
1. `'pipe'` - 在父子进程间建立一个管道。管道的父进程端作为`child_process`对象的一个属性暴露给父进程，比如`ChildProcess.stdio[fd]`。为fds 0到2 建立的管道也可以分别通过ChildProcess.stdin、ChildProcess.stdout和ChildProcess.stderr访问。
2. `ipc` - 在父子进程间建立一个IPC频道用来发送消息或者文件标识符。一个子进程最多有一个IPC stdio文件标识符。设置了这个选项就可以使用ChildProcess.send()方法。如果子进程向这个文件标识符写入JSON消息，这将会触发ChildProcess.on('message')。如果子进程是一个Node.js程序，那么IPC通道的存在会激活process.send()和process.on('message')。
3. `ignore` - 不要再子进程中设置这个文件标识符。需要注意Node总是会为它生成的进程打开fs 0到2。如果这其中任何一个被忽略了，node会打开`/dev/null`并将它附给子进程的fd。
4. `Stream`对象 - 和子进程共享一个与tty，文件，socket或者一个管道有关的可读或可写流。该流底层的文件标识在子进程中会被复制给stdio数组索引对应的文件标识符。注意该流必须有一个底层标识符（在`'open'`事件出现之前文件流没有）。
5. 正数 - 该整数值被解释为父进程中当前打开的一个文件标识符。它与子进程共享，和`Stream`被共享的方式相似。
6. `null`，`undefined` - 使用默认值。对于stdio fd 0，1和2（也就是stdin, stdout, stderr），会创建一个管道，对于fd 3或者更高的，默认值是`'ignore'`。

例子：

	var spawn = require('child_process').spawn;
	
	// Child will use parent's stdios
	spawn('prg', [], { stdio: 'inherit' });
	
	// Spawn child sharing only stderr
	spawn('prg', [], { stdio: ['pipe', 'pipe', process.stderr] });
	
	// Open an extra fd=4, to interact with programs present a
	// startd-style interface.
	spawn('prg', [], { stdio: ['pipe', null, null, null, 'pipe'] });
	
####options.detached
如果设置了`detached`选项，子进程将会成为一个新的进程组的组长。这样的话子进程可以在父进程退出后继续运行。

默认情况下，父进程会等待脱离了的子进程退出。要阻止父进程等待给定的子进程，使用`child.unref()`方法，这样父进程的事件轮询在它的引用计数中讲不会包括子进程。

解绑一个长时间运行的进程并将它的输出写入一个文件的例子：

	var fs = require('fs'),
	    spawn = require('child_process').spawn,
	    out = fs.openSync('./out.log', 'a'),
	    err = fs.openSync('./out.log', 'a');
		
	var child = spawn('prg', [], {
	  detached: true,
	  stdio: [ 'ignore', out, err ]
	});
	
	child.unref();
	
当时用`detached`的选项来创建一个长时间的进程时，该进程不会在后台保持运行，除非向它提供一个不连接到父进程的stdio配置。如果继承了父进程的stdio，子进程将会继续附着在控制终端。

####options.customFds
这是一个已废弃的选项，叫做`customFds`，允许指定特殊的文件描述符作为子进程的stdio。这个API无法移植到所有平台，因此被去除。有了`customFds`就可以将新进程的`[stdin, stdout, stderr]`钩到现有的流上；-1表示创建新流。自己承担使用风险。

####child_process.exec(command[, options], callback)