## Node.js 常用模块简介

### 模块

在 Node 环境中，每一个 js 文件都是一个模块。每一个模块都可以将某些变量（包括函数）暴露出去，其他模块可以引用这个模块暴露出来的变量。

	module.exports = variable;

	var foo = require('./moduleName');

这样 `foo` 就会是另一个模块中想要暴露出来的变量了。

_实际实现_：
1. 每一个模块被 Node 做成了一个闭包，因此每一个模块中的所有变量都是在一个函数之内的。因为 JS 的定义域是以函数为基础的。因此在两个模块中声明同名的两个全局变量互不影响。
2. exports 的具体实现是每一个模块首先要初始化一个对象，这其中有一个键为 `exports`，其对应的值为一个对象，里面的东西是将要暴露出来的变量的引用。如果要加载某一个模块，那么加载完毕之后会 `return` 这个模块的 `exports` 对象，这样在加载完毕之后就可以拿到想要暴露出来的变量了。

### module: fs

`fs` 模块内置的都是和读写文件相关的函数，因此在调用这些函数之前首先要加一句 `var fs = require('fs');` ，其中的每一种函数都有异步和同步两种。在这里只介绍异步函数，因为在服务器端需被反复调用的代码必须为异步。

- `fs.readFile('filename', 'charset', function (err, data) {...});` 这里读的是一个文本文件
- `fs.readFile('filename', function (err, data) {...})` 这里读的是一个二进制文件，因此没有编码格式参数。在这里，`data` 是一个 `Buffer` 对象，本质是一个_字节数组_，可以使用 `Buffer(data, 'utf-8')` 来将一个字符串转化为 `Buffer` 对象
- `fs.writeFile('filename', data, function (err) {...});` 在这里的 `data` 如果是字符串，那么默认编码是 `utf-8`，如果是 `Buffer` 对象，那么写入的是二进制文件
- `fs.stat('filename', function (err, stat) {...})` 这个函数是读取一些文件的信息。`stat` 对象有一些现成的函数和属性
	- `stat.isFile()`
		- `stat.isDirectory()`
		- `stat.size`
		- `stat.birthtime`
		- `stat.mtime` 上次更改时间

### module: stream

在使用文件流的过程中，需要首先创建一个 `fs` 对象，然后再进行读、写流的操作

- `var rs = fs.createReadStream('sample.txt', 'utf-8')` 这就打开了一个读取流，然后可以让这个流_监听_一些时间，分别有
	- `rs.on('data', function (chunk) {...});` 在这里的 `chunk` 是数据块，因为数据流的体积可能很大，因此需要分为一个个的 `chunk` 
	- `rs.on('end', function () {...});` 监听到文件读取流完成
	- `rs.on('error', function (err) {...});` 监听到读取文件流时出错
- `var ws = fs.createWriteStream('output.txt', 'utf-8');` 打开了一个以 `utf-8` 格式编码的文件写入流
	- `ws.write('string');`
	- `ws.end();`
- `var ws = fs.createWriteStream('output.txt')` 这个写入流并未指定编码格式，因此写入的是二进制流
	- `ws.write(new Buffer('string', 'utf-8'));`
	- `ws.end();`
- `rs.pipe(ws)` 这个操作和 `linux` 下的 `|` 这个指令很像，是一个“管道”操作，这样相当于将一个文件读取流的结果传递给了文件写入流，因此这是一个文件的复制操作。

### module: http, url, path

在这里先贴一段代码来演示最简单的一个 http 服务器以示范 Node 对 HTTP 请求和响应的封装

	'use strict';
	 
	 var http = require('http');
	 
	 var server = http.createServer(function (request, response) {
		 console.log(request.method + ': ' + request.url);
		 response.writeHead(200, {'Content-Type': 'text/html'});
		 response.end('<h1>Hello, world!</h1>');
	 });
	 
	 server.listen(8080);
	 console.log('Server is running at http://127.0.0.1:8080/');

这一段代码演示的是不论给的是什么请求都会返回一个 html 字符串

`var url = require('url')` 这个模块可以使用 `parse()` 函数最后解析出来一个 `url` 对象

`var path = require('path')` 这个模块可以处理和构造目录

接下来示范一个前面所有模块的综合应用，可以通过 URL 读取指定文件并且最后返回一个静态页面。

	'use strict';
	 
	 var
		 fs = require('fs');
		 url = require('url');
		 path = require('path');
		 http = require('http');
	 
	 //通过命令行获取root目录，如果没有通过命令行去传那么就默认是当前目录
	 var root = path.resolve(argv[2] || '.');
	 
	 console.log('Static root dir: ' + root);
	 
	 var server = http.createServer(function (request, response) {
		 //解析一下要请求的文件的名字
		 var pathname = url.parse(request.url).pathname;
		 //拼接目录
		 var filepath = path.join(root, pathname);
		 fs.stat(filepath, function (err, stats) {
			 if (!err && stats.isFile()) {
				 console.log('200 ' + request.url);
				 response.writeHead(200);
				 fs.createReadStream(filepath).pipe(response);
			 } else {
				 console.log('404 ' + request.url);
				 response.writeHead(404);
				 response.end('404 Not Found');
			 }
		 });
	 });
	 
	 server.listen(8080);
	 console.log('Server is running at http://127.0.0.1:8080/');

如果我们在之前写好的一个静态页面工程下面跑这段代码，就应该这样写命令行来启动服务器到指定目录

`node file_server.js /Users/ningyu/Documents/GitHub/ife-baidu-2017/project_1/task_7`

这样请求 `http://localhost:8080/index.html` 就可以正常显示之前写好的页面了。

### module: crypto

该模块内置了用 C/C++ 实现的加解密模块，因此比用纯 JS 写出来的加解密函数要快很多（为什么写上次大作业不用 Node.js ！！！）

	const crypto = require('crypto');
	const hash = crypto.createHash('md5') //还有很多内置的hash函数
	hash.update('string or Buffre');

	const hmac = crepto.createHmac('sha256' ,'salt');

在这里的 `hmac()` 函数可以接受第二个参数，不同的参数值可以造成同一个算法，同一段文字的哈希值不同

还有提供了 AES 支持，但是需要自己封装函数，封装代码这里略过。

这个模块还提供了基于非对称密钥的密钥协商…


