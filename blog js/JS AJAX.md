## JS AJAX

### AJAX
在 JS 中，因为直接使用表单来提交数据会跳转到一个新的页面，并且网络不好时会产生404错误，因此可以使用 JS 来提交数据，并且用请求回来的数据更新 HTML 页面，这也叫作 AJAX 请求，具体代码如下

	function success(text) \{
	var textarea = document.getElementById('test-response-text');
	textarea.value = text;
	}
	
	function fail(code) \{
	var textarea = document.getElementById('test-response-text');
	textarea.value = 'Error code: ' + code;
	}
	
	var request = new XMLHttpRequest(); // 新建XMLHttpRequest对象
	
	request.onreadystatechange = function () \{ // 状态发生变化时，函数被回调
	if (request.readyState === 4) \{ // 成功完成
	// 判断响应结果:
	if (request.status === 200) \{
	// 成功，通过responseText拿到响应的文本:
	return success(request.responseText);
	} else \{
	// 失败，根据响应码判断失败原因:
	return fail(request.status);
	}
	} else \{
	// HTTP请求还在继续...
	}
	}
	
	// 发送请求:
	request.open('GET', '/api/categories');
	request.send();
	
	alert('请求已发送，请等待响应…');

这其中，onreadystatechange 是一个回调函数，其作用就是先声明并且定义，在满足条件时调用这个函数。open 函数有三个参数，第一个指定请求方式，第二个指定 URL ，第三个为是否使用异步，默认为 true，因此第三个参数一般无需指定，否则会陷入_假死_状态，整个页面会卡住。

### 跨域请求
值得注意的一点是，在这里的 URL 使用的是_相对路径_，当使用一个不同的网址在 console 界面一定会报错，这是因为 JS 在 AJAX 请求中_不允许进行跨域请求_。现在解决跨域请求的方式为 JSONP，即在 `head` 标签内新增一个 `<script>` 节点，并且将其属性 `src='xxx'` 为你想要的并且提供 API 接口的网站。一般这类型的网站都会传回一个它定义好的函数和参数，因此需要自己在 JS 中写好这个回调函数。

回调函数：
	function refreshPrice(data) {
	var p = document.getElementById('test-jsonp');
	p.innerHTML = '当前价格：' +
	data['0000001'].name +': ' + 
	data['0000001'].price + '；' +
	data['1399001'].name + ': ' +
	data['1399001'].price;
	}

触发函数：
	function getPrice() {
	var
	js = document.createElement('script'),
	head = document.getElementsByTagName('head')[0];
	js.src = 'http://api.money.126.net/data/feed/0000001,1399001?callback=refreshPrice';
	head.appendChild(js);
	}

可以看到触发函数的 URL 中有一个 `callback=refreshPrice` 这就是一开始我们写好的回调函数。这样就可以实现 AJAX 的跨域加载。

### HTML5 跨域请求
又称为 CORS ，其实现的原理为，自己的浏览器在向服务器发出请求时，包头中有一个 `origin=my.com` ，对方收到请求后返回数据时，包头中有一个 `Access-Control-Allow-Origin=my.com` 或者是 `*` ，这样说明服务器允许作为跨域请求的服务器。