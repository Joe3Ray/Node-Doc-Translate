##控制台
===

	Stability: `4` - API冻结

* 对象

用于向stdout或者stderr打印字符。与大多数浏览器提供的控制台对象功能类似，只不过这里是输出到stdout或者stderr。

当目标是一个终端或者文件时，控制台函数是同步的（以防过早退出时丢失信息）。当输出目标是一个管道时它是异步的（防止长时间阻塞）。

也就是说，在下面的例子中，stdout 是非阻塞的，而 stderr 则是阻塞的。

	$ node script.js 2> error.log | tee info.log

在日常使用中，你不需要太担心它是阻塞还是非阻塞的，除非你要记录大量数据。

###console.log([data][, ...])

向stdout打印并新起一行。这个函数可以像`printf()`一样接收多个参数。比如：

	var count = 5;
	console.log('count: %d', count);
	// prints 'count: 5'
	
如果在第一个字符串中没有找到格式化元素，那么 util.inspect 将被应用到各个参数。详见`util.format()`。

###console.info([data][, ...])

和`console.log`一样。

###console.error([data][, ...])

和`console.log`一样，但是它向stderr打印。

###console.warn([data][, ...])

和`console.error`一样。

###console.dir(obj[, options])

在`obj`上使用`util.inspect`然后把结果字符串打印到stdout。这个函数会忽略`obj`上任何自定义的`inspect()`方法。可以传递一个可选的选项对象来改变格式化字符串的某些方面：
* `showHidden` - 如果设置为`true`那么对象的非枚举的属性也将被显示。默认是`false`。
* `depth` - 告诉`inspect`函数在格式化对象时需要递归多少次。这在检查大而复杂的对象时很有用。默认是2。传递`null`值可以让他无限递归。
* `colors` - 如果设置为`true`，输出将会用ANSI颜色码设置样式。默认是`false`。颜色是可定制的，见下方。

###console.time(label)

标记一个时间。

###console.timeEnd(label)

结束定时器，记录输出。例如：

	console.time('100-elements');
	for (var i = 0; i < 100; i++) {
	  ;
	}
	console.timeEnd('100-elements');
	// prints 100-elements: 262ms
	
###console.trace(message[, ...])

向stderr打印`'Trace :'`，后面还有格式化的信息和当前位置的栈跟踪。

###console.assert(value[, message][, ...])

和`assert.ok()`类似，但是错误信息和用`util.format(message...)`格式化之后一样。