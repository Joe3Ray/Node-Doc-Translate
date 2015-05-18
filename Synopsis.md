##梗概
===
一个用Node写的Web服务器示例，返回"Hello World"：

	var http = require('http');
	
	http.createServer(function (request, response) {
	    response.writeHead(200, {'Content-Type': 'text/plain'});
	    response.end('Hello World\n');
	}).listen(8124);
	
	console.log('Server running at http://127.0.0.1:8124/');
	
要运行这个服务器，需要把代码放到一个叫`example.js`的文件中然后使用node执行它

	> node example.js
	Server running at http://127.0.0.1:8124/
	
文档中所有的例子都可以按照相似的方式运行。