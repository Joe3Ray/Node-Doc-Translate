##Buffer
===

	稳定性：3 - 稳定的
	
纯净的Javascript对Unicode友好但是无法很好的处理二进制数据。处理TCP流或者文件流时，处理字节流是必须的。Node有几种操作、创建和消费字节流的策略。

原始数据存储在`Buffer`类的实例中。一个`Buffer`实例类似于一个整数数组，但是对应这V8堆之外的一块原始内存分配区域。一个`Buffer`的大小不可改变。

`Buffer`类是全局的，是一个比较罕见的不需要`require('Buffer')`就可以使用的类。

在Buffers和JavaScript String之间转化时需要一个明确的编码方法。下面是不同的字符串编码。

* `'ascii'` - 只适用于7位的ASCII数据。这个编码方法非常快速，如果设置了高位将会被剥离。
* `'utf8'` - 多字节编码的Unicode字符。许多网页和其他文档形式使用UTF-8。
* `'utf16le'` - 2或4字节，little endian(LE)编码的Unicode字符。支持代理对（U+10000 to U+10FFFF）。
* `'ucs2'` - `'utf16le'`的别名。
* `'base64'` - Base64字符串编码。
* `'binary'` - 一个将原始的二进制数据编码为字符串的方法，仅使用每个字符的前8位。这个编码方法是被反对的，应该避免使用，尽可能使用Buffer对象。这个编码在以后的Node版本中会被移除。
* `'hex'` - 把每一个字节编码为2个16进制字符。

从一个Buffer创建一个类型数组需要注意下列事项：

1. buffer的内存是复制的，不是共享的。
2. buffer的内存被解释成一个数组，而不是一个字节数组。这表示，`new Uint32Array(new Buffer([1, 2, 3, 4]))`创建的是一个拥有4个元素的`Uint32Array`，四个元素是`[1, 2, 3, 4]`，不是只有一个`[0x1020304]`或者`[0x4030201]`元素的`Uint32Array`。

注意：Node.js v0.8在`array.buffer`中简单地保存了一个指向buffer的引用而不是复制它。

为了更高效，它引入了一个与类型数组规范有着细微不兼容的特性。`ArrayBuffer#slice()`复制了一个片段，但是`Buffer#slice()`创建了一个视图。

###Class: Buffer
Buffer类是一个直接处理二进制数据的全局类型。有很多方式可以构建它。

####new Buffer(size)
* `size` 数字类型

分配一个新的`size`字节的buffer。注意，`size`不能比`kMaxLength`大。否则会抛出一个`RangeError`。

####new Buffer(array)
* `array` 数组类型

使用8位字节数组分配一个新的buffer。

####new Buffer(buffer)
* `buffer` Buffer

把传过来的buffer数据复制到一个新的Buffer实例上。

####new Buffer(str[, encoding])
* `str` 字符串类型 - 需要编码的字符串
* `encoding` 字符串类型 - 需要使用的编码，可选

分配一个包含给定`str`的新buffer。`encoding`默认为`'utf8'`。

####类方法：Buffer.isEncoding(encoding)
* `encoding` 字符串类型 要测试的编码字符串

如果`encoding`是一个合法的编码参数，返回true，否则返回false

####类方法：Buffer.isBuffer(obj)
* `obj` 对象
* 返回值：布尔类型

测试一个对象是否是Buffer。

####类方法：Buffer.byteLength(string[, encoding])
* `string` 字符串类型
* `encoding` 字符串类型，可选，默认值：'utf8'
* 返回值：数字

给出一个字符串的真实字符长度。`encoding`默认是`'utf8'`。这和`String.prototype.length`不同，因为那个返回值是字符串中字符的个数。 

示例：

	str = '\u00bd + \u00bc = \u00be';
	
	console.log(str + ": " + str.length + " characters, " + Buffer.byteLength(str, 'utf8') + " bytes");
	
	// ½ + ¼ = ¾: 9 characters, 12 bytes
	
####类方法：Buffer.concat(list[, totalLength])
* `list` 数组类型 需要连接的Buffer对象的列表
* `totalLength` 数字 连接后这些buffers的总长度

返回列表中所有buffer连接后的结果。

如果列表中没有元素，或者totalLength设置成0，它将返回一个零长度的buffer。

如果列表中只有一个元素，将会返回列表中的第一个元素。

如果列表中不止一个元素，则会创建一个新的Buffer对象。

如果没有提供totalLength，它将从列表中的buffers读取。但是这会增加额外的循环，所以显示地指定长度运行速度更快。

####类方法：Buffer.compare(buf1, buf2)
* `buf1` Buffer
* `buf2` Buffer

和`buf1.compare(buf2)`一样。在对Buffers的数组进行排序的时候很有用：

	var arr = [Buffer('1234'), Buffer('0123')];
	arr.sort(Buffer.compare);

####buf.length