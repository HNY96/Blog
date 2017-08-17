## Node.js 之 Koa, Nunjucks, MVC

### koa2

koa2 是一个基于 Node.js 的 web 开发框架，其[官网](http://koa.bootcss.com/)可以查询文档。

在这里首先要**注意**的一点是，npm 安装的模块如果不加参数，仅仅是 `npm install module_name` 是只针对某一个项目而言的，其他项目还想用这些模块的话需要重新进行安装。并且，`node_modules` 这个文件夹不应该出现在版本控制中。

一个基于 koa 的服务器如下所示

	//导入的是一个class因此用大写开头
	const Koa = require('koa');
	//将其实例化为一个app
	const app = new Koa();
	 
	//对于任何一个请求，app将调用改异步函数处理请求
	app.use(async (ctx, next) => {
		//进行下一个异步函数
			await next();
			ctx.response.type = 'text/html';
			ctx.response.body = '<h1>Hello, koa2</h2>';
	});
	//监听3000端口
	app.listen(3000);
	console.log('app started at port 3000...');

在这里比较困惑的一点是关键字 `async`，这个参数表明了对于每一个请求，koa 将调用我们传入的异步函数来处理。

`ctx` 是一个封装了 request 和 response 的变量，可以通过它来访问 request 和 response，`next()` 是传入的下一个异步函数，**也就是下一个`app.use()`**。这里涉及到了 koa 涉及的一个核心，即 _middleware_，中文称其为_中间件级联_。每一个 async 函数内部都可以做自己所必须要做的事情，其他事情只需要在这个函数里写一句 `await next()` ，再在下一个 async 函数中将功能完善即可。

如果一个 middleware 函数中没有调用 `await next()` 就会造成链的断裂，无法执行后续的 middleware 。因此可以依赖这个特性来检测用户权限，以此来确定是否还要执行后续代码。

运行以上代码会发现，只要在3000端口，只要以 get 访问，不论传入什么参数都会返回一个 `Hello, koa2` ，这样看起来很蠢，因此需要对不同的 url 做路由。因此需要引入一个新的 module

### koa-router

安装完成后，需要这样用

	//直接调用一个函数
	const router = require('koa-router')();
	
	router.get('url', async fn(ctx, next) {...});
	router.get('url2', async fn(ctx, next) {...});
	
	//注册为 middleware
	app.use(router.routes());
	...

这样就可以使用这个 module 来处理不同的 GET 请求。为了处理 POST 请求，就需要解析 request 的 body 部分，但是不论是 Node.js 还是 koa 本身都不提供解析的功能，因此需要引入另一个模块。

### koa-bodyparser

	const bodyParser = require('koa-bodyparser');
	
	//在合适的地方，即在 router 注册到 app 之前
	app.use(bodyParser());

这样，就会将 POST 中的参数放到 `ctx.request.body` 中。下面是一个使用 GET 和 POST 的例子

	router.get('/', async (ctx, next) => {
		ctx.response.body = `<h1>Index</h1>
		 <form action="/signin" method="post">
			 <p>Name: <input name="name" value="koa"></p>
			 <p>Password: <input name="password" type="password"></p>
			 <p><input type="submit" value="Submit"></p>
		 </form>`;
	});
	 
	router.post('/signin', async (ctx, next) => {
		 var
			 name = ctx.request.body.name || '',
			 password = ctx.request.body.password || '';
		 console.log('signin with name: ${name}, password: ${password});
		 if (name === 'koa' && password === '12345') {
			 ctx.response.body = '<h1>welcome, ${name}</h1>';
		 } else {
			 ctx.response.body = `<h1>Login failed</h1>
			<p><a href='/'>Try again</a></p>`;
		}
	});

### 重构

随着需要处理的 URL 越来越多，会伴随着这个文件越来越长，而且变得越来越难以维护，需要进行重构。

1. 将每一个 `router` 的第二个参数的那个函数抽象出来，写成 `var fn = async (ctx, next) => {...}`。将这个页面命名为 `index.html` 放在 `controllers` 文件夹下，最后通过 `module.exports = {'url': fn};` 将其暴露出来
2. 修改 `app.js` 让其扫描 `controllers` 目录，找到所有的 `js` 文件，将其导入然后注册

### Nunjucks

使用一个模板引擎是非常简单的，因为本质上只需要构造这样一个函数：

	function render (view, model) {
			...
	}

在这里，`view` 是模板的名称—**视图**，`model` 是数据，这个函数返回一个字符串，也就是模板的输出。

#### 特性一：
在使用 nunjucks 之后，需要在 `app.js` 中有一段固定的代码，然后就可以使用这个函数 `var s = env.render('htmlpage', {});`，这个视图模板存放在 `views` 文件夹下，第二个参数是一个 `Object`，重要的一点是，__在数据中的所有字符串均可以自动转义，不会造成 XSS 攻击__

#### 特性二：
此模块提供了强大的 tag ，编写条件判断、循环等功能，其[文档](http://mozilla.github.io/nunjucks/cn/templating.html)提供中文参考。

#### 特性三：
模板之间可以互相继承。如果有一个网站的很多子页是非常相似的，如果想要更改头和尾只需要更改模板，而无需更改所有文件。

一般而言，会先定义一个基本的网页框架 `base.html`

	<body>
	{% block header %} <h3>Unnamed</h3> {% endblock %}
	{% block body %} <div>No body</div> {% endblock %}
	{% block footer %} <div>copyright</div> {% endblock %}
	</body>

子模板 `extend.html`

	{% extends 'base.html' %}
	{% block header %}<h1>{{ header }}</h1>{% endblock %}
	{% block body %}<p>{{ body }}</p>{% endblock %}

然后进行渲染

	env.render('extend.html', {header: 'Hello', body: '...',});

就会发现，只有 `header` 和 `body` 发生了变化，`footer` 并未变。

### MVC

> MVC：Model-View-Controller

这一节实现了一个简单的 url 的映射以及网页的处理，详见本人 fork 下来的[github](https://github.com/HNY96/learn-javascript/tree/master/samples/node/web/koa/view-koa)



