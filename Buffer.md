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
* 数字

这个buffer的字节数。注意这可能不是内容的大小。`length`指向为这个buffer对象分配的内存大小。它不会随着buffer内容大小的变化而变化。

	buf = new Buffer(1234);
	
	console.log(buf.length);
	buf.write("some string", 0, "ascii");
	console.log(buf.length);
	
	//1234
	//1234
	
`length`属性是不可改变的，改变`length`的值可能会引起不可预知的反常表现。想要改变buffer的长度的应用应该把`length`视为只读属性，转而使用`buffer.slice`来创建一个新的buffer。

	buf = new Buffer(10);
	buf.write("abcdefghj", 0, "ascii");
	console.log(buf.length);    // 10
	buf = buf.slice(0, 5);
	console.log(buf.length);    // 5
	
####buf.write(string[, offset][, length][, encoding])
* `string` 字符串 - 需要写入buffer的数据
* `offset` 数字，可选，默认值：0
* `length` 数字，可选，默认值：`buf.length - offset`
* `encoding` 字符串，可选，默认值：'utf8'

在`offset`的位置使用指定的编码把`string`写入buffer。`offset`默认值为0，`encoding`默认值为`'utf8'`。`length`代表要写入的字节长度。返回值为写入的8位字节长度。如果buffer没有足够的空间来匹配整个字符串，它将写入其中的一部分。`length`默认值为`buffer.length - offset`。这个函数不会只写入一部分字符。

	buf = new Buffer(256);
	len = buf.write('\u00bd + \u00bc = \u00be', 0);
	console.log(len + " bytes: " + buf.toString('utf8', 0, len));
	
####buf.writeUIntLE(value, offset, byteLength[, noAssert])

####buf.writeUIntBE(value, offset, byteLength[, noAssert])

####buf.writeIntLE(value, offset, byteLength[, noAssert])

####buf.writeIntBE(value, offset, byteLength[, noAssert])
* `value` {数字} 需要写入buffer的字节
* `offset` {数字} `0 <= offset <= buf.length`
* `byteLength` {数字} `0 < byteLength <= 6`
* `noAssert` {布尔值} 默认值：false
* 返回：{数字}

根据指定的`offset`和`byteLength`将`value`写入buffer。支持高达48位的精度。例如：

	var b = new Buffer(6);
	b.writeUIntBE(0x1234567890ab, 0, 6);
	// <Buffer 12 34 56 78 90 ab>
	
将`noAssert`设置为`true`可以跳过对`value`和`offset`的合法性检测。默认值`false`。

####buf.readUIntLE(offset, byteLength[, noAssert])

####buf.readUIntBE(offset, byteLength[, noAssert])

####buf.readIntLE(offset, byteLength[, noAssert])

####buf.readIntBE(offset, byteLength[, noAssert])
* `offset` {数字} `0 <= offset <= buf.length`
* `byteLength` {数字} `0 < byteLength <= 6`
* `noAssert` {布尔型} 默认值：false
* 返回：{数字}

所有数字读取方法的通用版本。支持高达48位的精度。例如：

	var b = new Buffer(6);
	b.writeUint16LE(0x90ab, 0);
	b.writeUInt32LE(0x12345678, 2);
	b.readUIntLE(0, 6).toString(16);   // Specify 6 bytes (48 bits)
	// output: '1234567890ab'
	
将`noAssert`设置为`true`可以调过对`offset`的合法性检测。这意味着`offset`可能会超出buffer的范围。默认值为`false`。

####buf.toString([encoding][, start][, end])
* `encoding` 字符串，可选，默认值：'utf8'
* `start` 数字，可选，默认值：0
* `end` 数字，可选，默认值：`buf.length`

根据指定的encoding返回一个解码后的字符串。如果`encoding`是`undefined`或者`null`，则使用默认值`'utf8'`。start和end参数默认值分别为0和buffer.length。

	buf = new Buffer(26);
	for (var i = 0 ; i < 26 ; i++) {
	    buf[i] = i + 97; // 97 is ASCII a
	}
	buf.toString('ascii'); // outputs: abcdefghijklmnopqrstuvwxyz
	buf.toString('ascii',0,5); // outputs: abcde
	buf.toString('utf8',0,5); // outputs: abcde
	buf.toString(undefined,0,5); // encoding defaults to 'utf8', outputs abcde
	
参考上面`buffer.write()`的例子。

####buf.toJSON()
返回一个用JSON表示的Buffer实例。`JSON.stringify`在把一个Buffer实例转化为字符串时隐式地调用了这个函数。

例子：

	var buf = new Buffer('test');
	var json = JSON.stringify(buf);
	
	console.log(json);
	// '{"type":"Buffer","data":[116,101,115,116]}'
	
	var copy = JSON.parse(json, function (key, value) {
	    return value && value.type === 'Buffer'
	        ? new Buffer(value.data)
	        : value;
	});
	
	console.log(copy);
	// <Buffer 74 65 73 74>
	
####buf[index]
获取和设置第`index`位的8位字节。这个值代表单独的一个字节，所以合法的范围是16进制的`0x00`到`0xFF`或者`0`到`255`。

示例：逐字节的复制一个ASCII字符串到一个buffer：

	str = "node.js";
	buf = new Buffer(str.length);
	
	for (var i = 0; i < str.length; i++) {
	    buf[i] = str.charCodeAt(i);
	}
	
	console.log(buf);
	
	// node.js
	
####buf.equals(otherBuffer)
* `otherBuffer` Buffer

返回一个布尔值表示`buf`和`otherBuffer`是否具有相同的字节。

####buf.compare(otherBuffer)
* `otherBuffer` Buffer

返回一个数字用来表示在排序上`buf`在`otherBuffer`前面还是后面，或是两者一样。

####buf.copy(targetBuffer[, targetStart][, sourceStart][, sourceEnd])
* `targetBuffer` Buffer对象，需要复制进去的Buffer
* `targetStart` 数字，可选，默认值：0
* `sourceStart` 数字，可选，默认值：0
* `sourceEnd` 数字，可选，默认值：`buf.length`

从`buf`中复制一块区域的数据，写入到`targetBuffer`中的指定区域，即使他们有重叠的部分。如果没有给定`targetStart`、`sourceStart`或者`sourceEnd`，则使用它们的默认值，分别为`0`，`0`，`buf.length`

示例：创建两个Buffer，然后把`buf1`的第16到19位字节复制进`buf2`，从`buf2`的第8位字节处开始。

	buf1 = new Buffer(26);
	buf2 = new Buffer(27);
	
	for (var i = 0 ; i < 26 ; i++) {
	    buf1[i] = i + 97; // 97 is ASCII a
	    buf2[i] = 33; // ASCII !
	}
	
	buf1.copy(buf2, 8, 16, 20);
	console.log(buf2.toString('ascii', 0, 25));
	
	// !!!!!!!!qrst!!!!!!!!!!!!!
	
示例：创建一个单独的buffer，然后从中复制一块区域的数据写入同一个buffer的一块重叠区域

	buf = new Buffer(26);
	
	for (var i = 0 ; i < 26 ; i++) {
	    buf[i] = i + 97; // 97 is ASCII a
	}
	
	buf.copy(buf, 0, 4, 10);
	console.log(buf.toString());
	
	// efghijghijklmnopqrstuvwxyz
	
####buf.slice([start][, end])
* `start` 数字，可选，默认值：0
* `end` 数字，可选，默认值：`buffer.length`

返回一个新的buffer，但是和原来的buffer指向同一块内存，但是根据`start`（默认值`0`）和`end`（默认值`buffer.length`）的索引来进行偏移和截取。负数索引表示从buffer的末尾开始数。

**修改新的buffer会同步修改原来buffer的内存**

示例：用ASCII字母表创建一个Buffer，截取其中一段，然后修改原来buffer的一个字节。

	var buf1 = new Buffer(26);
	
	for (var i = 0; i < 26; i++) {
	    buf1[i] = i + 97; // 97 is ASCII a
	}
	
	var buf2 = buf1.slice(0, 3);
	console.log(buf2.toString('ascii', 0, buf2.length));
	buf1[0] = 33;
	console.log(buf2.toString('ascii', 0, buf2.length));
	
	// abc
	// !bc

####buf.readUInt8(offset[, noAssert])
* `offset` 数字
* `noAssert` 布尔值，可选，默认值：false
* 返回：数字

在指定偏移位置读取无符号的8位整数。

把`noAssert`设置为`true`可以跳过`offset`的有效性验证。这意味着`offset`的值可能超出buffer的范围。默认值是`false`。

例子：

	var buf = new Buffer(4);
	
	buf[0] = 0x3;
	buf[1] = 0x4;
	buf[2] = 0x23;
	buf[3] = 0x42;
	
	for (ii = 0; ii < buf.length; ii++) {
	    console.log(buf.readUInt8(ii));
	}
	
	// 0x3
	// 0x4
	// 0x23
	// 0x42
	
####buf.readUInt16LE(offset[, noAssert])

####buf.readUInt16BE(offset[, noAssert])
* `offset` 数字
*  `noAssert` 布尔值，可选，默认值：false
*  返回：数字

在指定偏移位置按照指定的endian格式读取一个无符号16位整数。

把`noAssert`设置为`true`可以跳过`offset`的有效性验证。这意味着`offset`的值可能超出buffer的范围。默认值是`false`。

例子：

	var buf = new Buffer(4);
	
	buf[0] = 0x3;
	buf[1] = 0x4;
	buf[2] = 0x23;
	buf[3] = 0x42;
	
	console.log(buf.readUInt16BE(0));
	console.log(buf.readUInt16LE(0));
	console.log(buf.readUInt16BE(1));
	console.log(buf.readUInt16LE(1));
	console.log(buf.readUInt16BE(2));
	console.log(buf.readUInt16LE(2));
	
	// 0x0304
	// 0x0403
	// 0x0423
	// 0x2304
	// 0x2342
	// 0x4223
	
####buf.readUInt32LE(offset[, noAssert])

####buf.readUInt32BE(offset[, noAssert])
* `offset` 数字
*  `noAssert` 布尔值，可选，默认值：false
*  返回：数字

在指定偏移位置按照指定的endian格式读取一个无符号32位整数。

把`noAssert`设置为`true`可以跳过`offset`的有效性验证。这意味着`offset`的值可能超出buffer的范围。默认值是`false`。

例子：

	var buf = new Buffer(4);
	
	buf[0] = 0x3;
	buf[1] = 0x4;
	buf[2] = 0x23;
	buf[3] = 0x42;
	
	console.log(buf.readUInt32BE(0));
	console.log(buf.readUInt32LE(0));
	
	// 0x03042342
	// 0x42230403
	
####buf.readInt8(offset[, noAssert])
* `offset` 数字
*  `noAssert` 布尔值，可选，默认值：false
*  返回：数字

在指定偏移位置读取有符号的8位整数。

把`noAssert`设置为`true`可以跳过`offset`的有效性验证。这意味着`offset`的值可能超出buffer的范围。默认值是`false`。

除了buffer内容被当做2的补符号值之外，和`buffer.readUInt8`作用一样。

####buf.readInt16LE(offset[, noAssert])

####buf.readInt16BE(offset[, noAssert])
* `offset` 数字
*  `noAssert` 布尔值，可选，默认值：false
*  返回：数字

在指定偏移位置按照指定的endian格式读取一个有符号16位整数。

把`noAssert`设置为`true`可以跳过`offset`的有效性验证。这意味着`offset`的值可能超出buffer的范围。默认值是`false`。

除了buffer内容被当做2的补符号值之外，和`buffer.readUInt16*`作用一样。

####buf.readInt32LE(offset[, noAssert])

####buf.readInt32BE(offset[, noAssert])
* `offset` 数字
*  `noAssert` 布尔值，可选，默认值：false
*  返回：数字

在指定偏移位置按照指定的endian格式读取一个有符号32位整数。

把`noAssert`设置为`true`可以跳过`offset`的有效性验证。这意味着`offset`的值可能超出buffer的范围。默认值是`false`。

除了buffer内容被当做2的补符号值之外，和`buffer.readUInt32*`作用一样。

####buf.readFloatLE(offset[, noAssert])

####buf.readFloatBE(offset[, noAssert])
