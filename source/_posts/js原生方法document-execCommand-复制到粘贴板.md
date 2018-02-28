---
title: js原生方法document.execCommand()复制到粘贴板
date: 2018-02-28 10:01:42
categories:
- javascipt
tags:
- javascript
- 原生方法
---
## `document.execCommand()`语法
`bool = document.execCommand(aCommandName, aShowDefaultUI, aValueArgument)` 返回值是布尔值，表示是否支持。

`aCommandName`: 命令名字,`copy,cut`等
`aShowDefaultUI`:一个 `Boolean`， 是否展示用户界面，一般为 false。
`aValueArgument`:一些命令（例如insertImage）需要额外的参数（insertImage需要提供插入image的url），默认为null,一般不用。
## 使用
### 从输入框复制
页面需要一个`input`标签，demo如下：
``` html 
<input id="demoInput" value="hello world">
<button id="btn">点我复制</button>
```
``` javascript
let btn = document.querySelector('#btn');
btn.addEventListener('click', () => {
	let input = document.querySelector('#demoInput');
	input.select();
	if (document.execCommand('copy')) {
		document.execCommand('copy');
		console.log('复制成功');
	}
})
```

### 不从输入框复制
<!--more-->
由于`document.execCommand()`只能操作可编辑区域，即`<input>`,`<textarea>`,这样就要动态添加可编辑区域再删除。
``` html
<div id="val">复制内容</div>
<button id="btn">点我复制</button>
```
``` javascript
let btn = document.querySelector('#btn');
let val = document.querySelector('#val').innerHTML;
btn.addEventListener('click',() => {
	let input = document.createElement('input');
  input.setAttribute('readonly', 'readonly');
  input.setAttribute('value', val);
  document.body.appendChild(input);
	input.setSelectionRange(0, val.length);
	if (document.execCommand('copy')) {
		document.execCommand('copy');
		console.log('复制成功');
	}
    document.body.removeChild(input);
})
```
`input.setAttribute('readonly', 'readonly')`,兼容在iOS环境下瞬间拉起键盘又缩回的闪烁；
`input.setSelectionRange(0, val.length);`,还是因为在iOS环境下，`input.select()`没有选中全部内容。