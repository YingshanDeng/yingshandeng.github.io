title: JS 中遍历树
date: 2017-02-26 16:43:46
tags: [递归遍历, 非递归深度优先遍历, 非递归广度优先遍历]
categories: 算法
---


本文回顾一下遍历树的几种方式：递归遍历，非递归深度优先遍历，非递归广度优先遍历。

<!-- more -->

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/D0A39C6C-2A93-4E2D-8FFD-C209BD563C69.png)
如上图，在 JavaScript 中可以用 object 表示如下：
```
let tree = {
    value: 0,
    children: [{
        value: 11,
        children: [{
        	value: 21,
        	children: [{
        		value: 31,
        		children: []
        	}, {
        		value: 32,
        		children: []
        	}, {
        		value: 33,
        		children: []
        	}]
        }, {
        	value: 22,
        	children: []
        }]
    }, {
        value: 12,
        children: [{
        	value: 23,
        	children: []
        }, {
        	value: 24,
        	children: []
        }]
    }, {
        value: 13,
        children: []
    }]
}
```

## 递归遍历
```
var recursiveTraverse = function (node, action) {
	if (!node || !node.children) { return; }
	action(node.value);
	node.children.forEach(function(item, index) {
		recursiveTraverse(item, action);
	});
}
// 递归实现
recursiveTraverse(tree, console.log);
```
输出结果：
0
11
21
31
32
33
22
12
23
24
13

## 非递归深度优先遍历（借助栈实现）
```
var nonRecursiveDepthFirstTraversal = function (node, action) {
	if (!node || !node.children) { return; }
	var _stack = []; // 借助一个栈
	_stack.unshift(node);

	while (_stack.length) {
		let _curNode = _stack.shift(); // 推出栈顶元素
		action(_curNode.value);
		// 将子节点依次放入到栈顶
		// _curNode.children.forEach(function (item, index) {
		// 	_stack.unshift(item);
		// })
		if (_curNode.children.length) {
			_stack = _curNode.children.concat(_stack);
		}
	}
}
// 非递归深度优先遍历
nonRecursiveDepthFirstTraversal(tree, console.log);
```
输出结果：
0
11
21
31
32
33
22
12
23
24
13

## 非递归广度优先遍历（借助队列实现）
```
var nonRecursiveWidthFirstTraversal = function (node, action) {
	if (!node || !node.children) { return; }
	var _queue = []; // 借助一个队列
	_queue.push(node);
	while (_queue.length) {
		let _curNode = _queue.shift(); // 推出队头元素
		action(_curNode.value);
		// 将子节点依次推入队列中
		// _curNode.children.forEach(function (item, index) {
		// 	_queue.push(item);
		// })
		if (_curNode.children.length) {
			_queue = _queue.concat(_curNode.children);
		}
	}
}
// 非递归宽度优先遍历
nonRecursiveWidthFirstTraversal(tree, console.log);
```
输出结果：
0
11
12
13
21
22
23
24
31
32
33