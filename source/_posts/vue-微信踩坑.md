---
title: vue 微信踩坑
date: 2018-02-08 11:31:24
categories:
- JavaScript
- 前端
- 微信
tags:
- javascript
- vue
- 微信授权
- 微信支付
---
1.  App分享到微信：
	* 分享之后微信会自动在`/#/`之后加上`?from=timeline`或`?from=groupmessage`或`?from=singlemessage`
	* 分享之后第一次链接打开跳转授权是正常的，第二次打开如果有多个参数就会被去掉第一个之后的所有参数
/解决办法/：
	1. 分享微信自动加参数可以用`vue-router`的前置路由守卫，代码实例
	分享链接比如`http://www.xxx.com/#/index?data=value1-value2`
``` javascript
router.beforeEach((to, from, next) => {
  if (to.query.from) {
    let queryString = to.query.from.split('=')[1];
    let nextQuery = {
      data: queryString
    };
    next({ path: '/booklist', query: nextQuery })
  } else {
    next();
  }
})	
```
<!--more-->
	2. app分享时只传一个参数，把所有参数值用`_`连接，比如`data=value1-value2`，不要用`key1=value1&key2=value2`，取值的时候可以用`this.$route.query.data.split('_')`得到数组之后再取单个的值。
```javascript
if (this.$route.query.data) {
  let queryArray = this.$route.query.data.split('_');
  this.key1 = queryArray[0];
  this.key2 = queryArray[1];
}	
```
2. vue微信问题
	* 微信获取授权
	如果使用的路由模式是默认hash，把`#`去掉，判断是否已经授权，如果未授权，用`window.location.href`跳转后台授权接口，并且把处理后的链接地址当作参数带过去，`window.location.href='/wxAuth?url='+url;`然后后台授权成功之后跳转中转页面，把页面链接url当作参数传过来，用再把`#`加上去，设置已授权`flag`，再跳转回原来的页面；
	访问页面：
```javascript
let url = window.location.href;
let isAuth = window.sessionStorage.getItem('isAuth');
if (!isAuth) {
  let nextUrl = url.replace('#/', '');
  window.location.href = "/wxAuth?url="+url;
} else {
	...
}	
```
	中转页面：
```javascript
let url = window.location.href;
let reUrl = window.location.search.split('?url=')[1];
window.sessionStorage.setItem('isAuth', true);
reUrl = reUrl.replace('index', '#/index');
window.location.href = reUrl;
```
	* 微信支付
	首先，`wx.config`中注入`chooseWXPay`微信支付后台授权目录配到二级，在支付页面`#`前面加上`?`,然后原页面跳转一下，
```javascript
let url = window.location.href;
if (!url.match(/\?#/)) {
  window.location.replace(window.location.href.split('#')[0] + '?' + window.location.hash);
      return;
 }
```
