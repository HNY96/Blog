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

