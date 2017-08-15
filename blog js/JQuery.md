## JQuery

JQuery （以下简称 JQ）是一个 JS 的库，提供了很多包装好的函数，只需在 HTML 头中加一句 `<script src='http://code.jquery.com/jquery-3.2.1.min.js'></script>` 即可从其官网上引用其资源。

### 选择器

JQ 中将选择器进行了抽象，因此有以下语法：

- `$('#id')`
- `$('.class')`
- `$('tag')`
- `$('[type=typename])`
- `$('some, other')`
- `$('ancestor descendant')`
- `$('parent>child')`
- `$('some:filter')` 这里面的 filter 是一些常用的，比如说 checked、radio、first-child、nth-child(num)

除去这些常用的选择器，还有一些函数辅助进行查找过滤操作：
- find() 这个函数是在找到一个 JQ 对象之后，对其使用 find 函数，其_参数_为想要满足的条件，可以使用前面介绍的选择器
- parent() next() prev() 顾名思义，后两个函数可以接受参数，并且返回第一个符合条件的
- filter() 其参数有_两种_：
	1. 即上文介绍的选择器
	2. 一个匿名函数，注意匿名函数里面的 this 是 DOM 对象而不是 JQ 对象，例如
			jq.filter( function() \{
				this.innerHTML.indexOf(’S’) === 0
			\}).get(); // 返回 S 开头的节点
- map() 和 python 的 map 几乎相同，其接受一个匿名函数作为一种映射规则
- fist() last() slice(start, end) 顾名思义，如果一个 JQ 对象包含多个，那么返回指定的那个

**小技巧**
在实际代码中发现，获得一个 input 的 JQ 对象之后，无需 `.get(0).value`，只要在 JQ 对象之后跟一个函数 `.val()` 即可

### 操作DOM

JQ 对象可以很方便的修改和获取 DOM，在这里列出以下函数

- text(), html() 可以获取一个 JQ 对象其对应的文本或者 HTML 语句，如果在其中包含了参数，那么可以修改对应的 DOM
- css('name', ['value']) 可以获取或者修改一个 DOM 节点的样式。由于其修改的是`style`属性，因此_优先级最高_
- hasClass() addClass() removeClass() 可以向一个节点判断是否有一个类，添加一个类，移除一个类。可以事先给这个类写好样式，然后一次更改多个属性
- hide() show() 无需考虑`display`属性原来是什么，只需要将其隐藏或者展示即可
- width() height() 可以获取当前 JQ 对象的宽和高，传入参数的话还可以修改其对应的属性
- attr('name',['value']) removeAttr('name') 获取或者修改某个属性的值，移除某个属性。如果该属性不存在，返回`undefined`
- prop('name') 和上一个类似，只不过这个是返回`true` or `false`
- is('pseudoSlector') 一般常用的场合是判断某个 DOM 是否包含某个_伪选择器_，返回`true` or `false`
- val() 获取其值，如果传参那么可以设置 `value` 属性
- append() prepend 父节点调用这个函数，向其子节点的最后/最前加一个参数传入的 JQ对象、DOM 对象或函数均可。
- after() before() 同上述的两个方法相似，只不过这个实在兄弟节点之间，是同级的关系
- remove() 移除选中的节点，_而非_其子节点

### 事件

#### 绑定
绑定事件给某些 JQ 对象被大大地化简，而无需繁琐的 `addEventListener`，在这里只需使用 `on('click', function() {});`即可绑定一个点击事件，当然对于点击而言有更简易的写法：`click(function)`。

`on()`函数可以绑定很多事件，具体如下：
- click, dbclick, mouseenter, mouseleave, mousemove, hover
- keydown, keyup, keypress **仅作用在获得焦点的`input``textarea`上**
- focus, blur, change, submit
- ready 这个相当于原来的 `window.onload()`只有在页面全部加载完成时才会调用`on`绑定的函数，一般有这么几种写法：

			$(document).on('ready', function () {
				$('#testForm).on('submit', function () {
					alert('submit!');
				});
			});
---- 
			$(document).ready(function () {
				// on('submit', function)也可以简化:
				$('#testForm).submit(function () {
					alert('submit!');
				});
			});
---- 
			$(function () {
				//init...
			});

其中以第三种写法最为常见，因此要掌握这种写法

#### 事件参数
	$('#testMouseMoveDiv').mousemove(function (e) {
		$('#testMouseMoveSpan').text('pageX = ' + e.pageX + ', pageY = ' + e.pageY);
	});
以上代码说明了_事件参数_，`mousemove()``keypress()`都会传入一个事件对象`event`，这个对象有多个属性，可以访问其属性来实现特定的功能

#### 取消绑定
取消绑定可以用函数`off`，但是如果`on`和`off`都用的是**匿名函数**，那么这两个匿名函数指向的是不同的函数，这样是不能解绑的。

除此之外，`off('event')`和`off()`是可以一次解绑所有的指定时间的事件，或者所有事件

### 动画

用 JQ 实现动画要比纯用 JS 方便许多，有许多现成的函数可供使用：
- show() hide() toggle() 每一个函数均可以传参，动画效果是在左上角收缩或者展开
- slideUp() slideDown() slideToggle() 传参同上，动画效果是上下滑动的
- fadeIn() fadeOut() 淡入淡出
- animate() 这个可以自定义动画效果，第一个参数是一个`Object`对象，里面和 CSS 的样式表是相同的，指明了某对象的动画结束时状态，要注意**非纯数字记得用字符串**；第二个参数是一个数字，指明了动画的执行时间；第三个参数可选，可以传入一个_函数_，当动画结束时该函数被执行
- delay() 传入参数后可以时_串行动画_时实现暂停
- 串行动画：可以选中一个对象后向其执行以下类型的代码：

	var div = $('#test-div');
	div.slideUp();
		    .delay(1000);
	    .animate({
		something;
	    });
	...

**注意**：`slideUp`是对高度进行变化，然而非`block`元素没有`height`属性，因此只能使用 CSS3 的`transition`来进行改变。因此 JQ 也非万能的:)

### AJAX

JQ 也封装了原来 JS 所用的 AJAX ，详情见[我的Github](https://github.com/HNY96/Blog/blob/master/blog%20js/JS%20AJAX.md)。因此在 JQ 中对于 AJAX 的限制和 JS 中是一样的，只不过封装的更为抽象和高级。具体的一些函数如下：

`$.ajax('url', [Object)`
URL 即为发出请求的那个网站，第二个参数是一个对象，这里面可以对一些常用的参数进行修改：

- `async` 是否进行异步请求，默认为`true`，一般无需更改，否则可能使浏览器陷入假死状态
- `method` 发送请求时的方法，默认为`GET`，可以修改为`POST``PUT`等
- `contentType` 发送 `POST`时的数据格式
- `data` 发送的数据，可以是字符串，数组或者是对象，其会根据发送方法的不同自动封装成合适的格式
- `headers` 发送额外的 HTTP 请求头，必须为对象
- `dataType` 接受的数据格式，可以设置为`html``xml``json``text`等

在发送完成之后，可以用_链式写法_处理各种回调
	var jqxhr = $.ajax('/api/categories', {
	    dataType: 'json'
	}).done(function (data) {
	    ajaxLog('成功, 收到的数据: ' + JSON.stringify(data));
	}).fail(function (xhr, status) {
	    ajaxLog('失败: ' + xhr.status + ', 原因: ' + status);
	}).always(function () {
	    ajaxLog('请求完成: 无论成功或失败都会调用');
	});

`$.get('url', [Object)`
由于 `GET` 请求的普遍性，因此被单独抽象成了一个函数。对象中放着想要发送的参数。还有一个 `$.post()` 大同小异，在这里不加赘述

`$.getJSON('url', [Object).done(function (data) {...});`
这个函数采用了默认的 `GET` 方法，然后对得到的数据进行进一步操作

由于 JS 对 AJAX 的安全限制，如要进行使用 JSONP 的跨域加载，那么需要在 `ajax()` 第二个参数中加一个 `jsonp:'callback'`

### 扩展

在编程中，如果有重复性的操作，那么第一想法便是将其抽象成一个函数。JQ 中也提供了这样一个接口，使得 coder 可以自定义一些方法。

	$.fn.highlight1 = function () {
		this.css('background-color', '...').css('color', '...');
		 return this
	 }

这是一个最基本的想法，其中至于为什么要 `return` 回来，是因为如果不这样的话那么这个函数的后面无法继续串行执行其他函数。

如果用户想要自定义强调时的背景色和字体颜色时：

	$.fn.highlight2 = function (option) {
		 var backgroundColor = option && option.backgroundColor || '...';
		 var color = option && option.color || '...';
		 this.css('background-color', backgroundColor).css('color', color);
		 return this;
	 }

这也是一种实现方法，但是又有一种问题出现了：

> 如果用户想要自定义默认的强调样式该怎么办？

于是就有了最终版本的代码

	$.fn.highlight = function(options) {
		 var opts = $.extend({}, $.fn.highlight.defaults, options);
		 this.css('background-color', opts.backgroundColor).css('color', opts.color);
		 return this;
	 }
	 
	 $.fn.highlight.defaults = {
		 backgroundColor: '...',
		 color: '...'
	 }

在这里的 `$.extend()` 函数会接受至少三个参数，第一个参数为 `target` ，即最后的参数聚集地。后面的参数都必须为 `Object` 格式，优先级是_越靠后的优先级越大_，因此将用户指定的默认样式放到系统默认的后面，这样如果用户传进来了 `opts` 参数，那么就会优先使用用户指定的样式。除此之外，用户还可以更改 `$.fn.highlight.defaults` ，这样就无需每次调用时都指定了。 

