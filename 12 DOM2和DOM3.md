# 12 DOM2和DOM3

## 12.1DOM变化

### 12.1.1 针对XML命名空间的变化

略

### 12.1.2 其他方面的变化

略

## 12.2样式

### 12.2.1 访问元素的样式

| CSS属性           | javascript属性          |
| --------------- | --------------------- |
| backgroud-image | style.backgroundImage |
| color           | style.color           |
| display         | style.display         |
| font-family     | style.fontFamily      |

#### DOM样式属性和方法


```js
//cssText
myDiv.style.cssText = "width: 25px;height: 100px;";
alert(myDiv.style.cssText);
```

```js
//length and item
for(var i = 0, len = myDiv.style.length; i < len; i++) {
	alert(myDiv.style[i]); //myDiv.style.item(i)
}
```

```js
//getPropertyValue
var prop, value, i, len;
for(var i = 0, len = myDiv.style.length; i < len; i++) {
	alert(myDiv.style[i]);
	value = myDiv.style.getPropertyValue(prop);
	alert(prop + ':' + value);
}
```

```js
//more message
var prop, value, i, len;
for(var i = 0, len = myDiv.style.length; i < len; i++) {
	alert(myDiv.style[i]);
	value = myDiv.style.getPropertyCSSValue(prop);
	alert(prop + ':' + value.cssText + '(' + value.cssValueType + ')');
}
```

```js
//removeProperty
myDiv.style.removeProperty('border');
```

#### 计算的样式

```js
//获取受层叠影响的样式信息
myDiv = document.getElementById('myDiv');
var myDivStyle = document.defaultView.getComputedStyle(myDiv, null); //第二个参数可以室伪元素，如“:after”;ie9+
alert(myDivStyle.width);
```

```js
var myDivStyle = document.defaultView.currentStyle(myDiv); //ie only
alert(myDivStyle.width);
```

### 12.2.2 操作样式表

略

### 12.2.3 元素大小

#### 偏移量

 ![0821](http://www.vastskycc.com/zb_users/upload/2016/08/201608271472285453239847.png)

```js
//offset
function getElementLeft(element) {
	var actualLeft = element.offsetLeft;
	var current = element.offsetParent;
	while(current !== null) {
		actualLeft += current.offsetLeft;
		current = current.offsetParent;
	}
	return actualLeft;
}

function getElementTop(element) {
	var actualTop = element.offsetTop;
	var current = element.offsetParent;
	while(current !== null) {
		actualTop += current.offsetTop;
		current = current.offsetParent;
	}
	return actualTop;
}
//这些属性为只读，每次访问都需要计算，注意性能
```

#### 客户区大小

 ![0821-2](http://www.vastskycc.com/zb_users/upload/2016/08/201608271472285548515540.png)

```js
//client

function getViewport() {
	if(document.compatMode == 'BackCompat') {
		return { //ie7-
			width: document.body.clientWidth,
			height: document.body.clientHeight
		};
	} else {
		return {
			width: document.documentElement.clientWidth,
			height: document.documentElement.clientHeight
		};

	}
}
```

#### 滚动大小

 ![0821-3](http://www.vastskycc.com/zb_users/upload/2016/08/201608271472285591408476.png)

```js
//文档总高度
var docHeight = Math.max(document.documentElement.scrollHeight, document.documentElement.clientHeight);
var docWidth = Math.max(document.documentElement.scrollWidth, document.documentElement.clientWidth);
```

```js
//重置滚动
function scrollToTop(element) {
	if(element.scrollTop != 0) {
		element.scrollTop = 0;
	}
}
```

#### 确定元素大小

```js
//确定元素大小
function getBoundingClientRect(element) {
	var scrollTop = document.documentElement.scrollTop;
	var scrollLeft = document.documentElement.scrollLeft;
	if(element.getBoundingClientRect) {
		if(typeof(arguments.callee.offset) != "number") {
			var scrollTop = document.documentElement.scrollTop;
			var temp = document.createElement('div');
			temp.style.cssText = 'position: absolute;left: 0;top: 0;';
			document.body.appendChild(temp);
			arguments.callee.offset = -temp.getBoundingClientRect().top - scrollTop;
			document.body.removeChild(temp);
			temp = null;
		};

		var rect = element.getBoundingClientRect();
		var offset = arguments.callee.offset;
		return {
			left: rect.left + offset,
			rigfht: rect.right + offset,
			top: rect.top + offset,
			bottom: rect.bottom + offset
		};
	} else {
		var actualLeft = getElementLeft(element);
		var actualTop = getElementTop(element);
		return {
			left: actualLeft - scrollLeft,
			right: actualLeft + element.offsetWidth - scrollLeft,
			top: actualTop - scrollTop,
			bottom: actualTop + element.offsetHeight - scrollTop
		}
	}
}
```

## 12.3 遍历

> 对DOM树的遍历为深度优先
###12.3.1 NodeIterator

```html
<div id="div1">
	<p><strong>hellow</strong> world
	</p>
	<ul>
		<li>l1</li>
		<li>l2</li>
		<li>l3</li>
	</ul>
</div>
```

```js
var div = document.getElementById('div1');
var iterator = document.createNodeIterator(div, NodeFilter.SHOW_ELEMENT, null, false); //ie9+
var node = iterator.nextNode();
while(node !== null) {
	console.log(node.tagName); //输出标签名
	node = iterator.nextNode();
}
```

```js
//只返回Li
var div = document.getElementById('div1');
var filter = function(node) {
	return node.tagName.toLowerCase() == 'li' ? NodeFilter.FILTER_ACCEPT : NodeFilter.FILTER_SKIP;
};
var iterator = document.createNodeIterator(div, NodeFilter.SHOW_ELEMENT, filter, false);
var node = iterator.nextNode();
while(node !== null) {
	console.log(node.tagName); //输出标签名
	node = iterator.nextNode();
}
```

### 12.3.2 TreeWalker

```js
var div = document.getElementById('div1');
var filter = function(node) {
	return node.tagName.toLowerCase() == 'li' ? NodeFilter.FILTER_ACCEPT : NodeFilter.FILTER_SKIP;
};
var walker = document.createTreeWalker(div, NodeFilter.SHOW_ELEMENT, filter, false);
var node = walker.nextNode();
while(node !== null) {
	console.log(node.tagName); //输出标签名
	node = walker.nextNode();
}
```

```js
var div = document.getElementById('div1');
var walker = document.createTreeWalker(div, NodeFilter.SHOW_ELEMENT, nulll, false); //ie9+
walker.firstChild(); //转到p
walker.nextSibling(); //转到ul
var node = walker.firstChild(); //转到第一个li
while(node !== null) {
	console.log(node.tagName); //输出标签名
	node = walker.nextSibling()
}
```
```js
//起点
var node = walker.nextNode();
alert(node === walker.currentNode); //true
walker.currentNode = document.body;
```
## 12.4 范围

###12.4.1 DOM中的范围

#### 用DOM范围实现简单选择

```html
<p id="p1"><b>hello</b> world</p>
```

```js
var range1 = document.createRange(); //ie9+
var range2 = document.createRange();
var p1 = document.getElementById('p1');
range1.selectNode(p1);
range2.selectNodeContents(p1);
```

 ![0821-4](http://www.vastskycc.com/zb_users/upload/2016/08/201608271472286059709484.png)

#### 用DOM范围实现复杂选择

```js
//模仿selectNode()selectNodeContents()
var range1 = document.createRange(); //ie9+
var range2 = document.createRange();
var p1 = document.getElementById('p1');
var p1Index = -1;
var i, len;
for(i = 0, len = p1.parentNode.childNodes.length; i < len; i++) {
	if(p1.parentNode.childNodes[i] == p1) {
		p1Index = i;
		break;
	}
}
range1.setStart(p1.parentNode, p1Index + 1); //ie not sup
range1.setEnd(p1.parentNode, p1Index + 1);
range2.setStart(p1, 0);
range2.setEnd(p1, p1.childNodes.length);
```

```js
var p1 = document.getElementById('p1'),
	helloNode = p1.firstChild,
	worldNode = p1.lastChild,
	range = document.createRange();
range.setStart(helloNode, 2);
range.setEnd(worldNode, 3);
```

 ![0821-5](http://www.vastskycc.com/zb_users/upload/2016/08/201608271472286096415131.png)

#### 操作DOM范围中的内容

![0822-1](http://www.vastskycc.com/zb_users/upload/2016/08/201608271472286122920042.png)

以前面的例子而言，在he之后自动补了个`</b>`和`<b>以闭合区间

```js
//delete
var p1 = document.getElementById('p1'),
	helloNode = p1.firstChild,
	worldNode = p1.lastChild,
	range = document.createRange();
range.setStart(helloNode, 2);
range.setEnd(worldNode, 3);
range.deleteContents();
```

```html
<--结果-->
<p id="p1"><b>he</b>rld</p>
```

```js
//extractContents
var p1 = document.getElementById('p1'),
	helloNode = p1.firstChild,
	worldNode = p1.lastChild,
	range = document.createRange();
range.setStart(helloNode, 2);
range.setEnd(worldNode, 3);
var fragment = range.extractContents();//
p1.parentNode.appendChild(fragment);
```

```js
//cloneContents
var p1 = document.getElementById('p1'),
	helloNode = p1.firstChild,
	worldNode = p1.lastChild,
	range = document.createRange();
range.setStart(helloNode, 2);
range.setEnd(worldNode, 3);
var fragment = range.cloneContents();
p1.parentNode.appendChild(fragment);
```

#### 插入DOM范围中的内容

```html
<span style="color: red;">Inserted text</span>
```

```js
var p1 = document.getElementById('p1'),
	helloNode = p1.firstChild.firstChild,
	worldNode = p1.lastChild,
	range = document.createRange();
range.setStart(helloNode, 2);
range.setEnd(worldNode, 3);
var span = document.createElement('span');
span.style.color = 'red';
span.appendChild(document.createTextNode('Inserted text'));
range.insertNode(span);
```

```html
<--结果-->
<p id="p1"><b>he<span style="color: red;">Inserted text</span>llo</b> world</p>
```

```js
var p1 = document.getElementById('p1'),
	helloNode = p1.firstChild.firstChild,
	worldNode = p1.lastChild,
	range = document.createRange();
range.selectNode(helloNode);
var span = document.createElement('span');
span.style.backgroundColor = 'yellow';
range.surroundContents(span);

```

```html
<--结果-->
<p id="p1"><b><span style="background-color: yellow;">hello</span></b> world</p>
```

#### 折叠DOM范围

```js
//折叠
range.collapse(true);//折叠到起点
range.collapse(false);//折叠到终点
console.log(range.collapsed);//输出true
```

![0822-3](http://www.vastskycc.com/zb_users/upload/2016/08/201608271472286254860860.png)



#### 比较DOM范围

```js
range.compareBoundaryPoints()
```

#### 复制DOM范围

```js
var newRange = range.cloneRange();
```

#### 清理DOM范围

```js
range.detach(); //从文档中分离
range = null; //解除引用
```

### 12.4.2 IE8及更早版本中的范围

```js
//ie8+
//<p id="p1"><b>hello</b> world</p>

//select hello
var range = document.body.createTextRange();
var found = range.findText('hello');
alert(found); //true
alert(range.text); //hello

var found = range.findText('hello');
var foundAgain = range.findText('hello', 1); //正值往前找，负值往后找

//selectNode
var range = document.body.createTextRange();
var p1 = document.getElementById('p1');
range.moveToElementText(p1);

//html
alert(range.htmlText);
//父节点
var ancestor = range.parentElement();

//ie复杂
range.moveStart('word', 2); //起点移动2个字符
range.moveEnd('character', 1); //起点移动1个字符
range.move('chrarcter', 5); //移动5个字符，范围的起点和终点就一样了，必须创建新选区

var range = document.body.createTextRange();
var p1 = document.getElementById('p1');
range.findText('hello');
range.pasteHTML('<em>wow</em>');
```





