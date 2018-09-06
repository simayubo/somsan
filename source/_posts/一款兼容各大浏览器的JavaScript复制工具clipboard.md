---
title: 一款兼容各大浏览器的JavaScript复制工具clipboard
tags:
  - JavaScript
  - 前端
categories:
  - JavaScript
abbrlink: 465ab45e
date: 2018-08-30 19:32:11
---

## 简介
clipboard.js是一款轻量级的实现复制文本到剪贴板功能的JavaScript插件，脱离falsh复制插件。通过该插件可以将输入框，文本域，DIV元素中的文本等文本内容复制到剪贴板中 
clipboard.js支持主流的浏览器：chrome 42+; Firefox 41+; IE 9+; opera 29+; Safari 10+; 
官网：[https://clipboardjs.com/][1]
下载地址：[https://github.com/zenorocha/clipboard.js/archive/master.zip][2]

## 使用
引入js文件
```html
<script src="dist/clipboard.min.js"></script>
```
html代码
```html
<span style="cursor: pointer;" class="_copy" data-uri="http://baidu.com">复制地址</span>
```
Js代码
```js
$('._copy').click(function () {
    var uri = $(this).attr('data-uri');
    var clipboard = new Clipboard('._copy', {
        text: function() {
            return uri;
        }
    });
    clipboard.on('success', function(e) {
        alert('复制成功');
    });
    clipboard.on('error', function(e) {
        alert('您的浏览器不支持复制，请手动复制！');
    });
});
```

  [1]: https://clipboardjs.com/
  [2]: https://github.com/zenorocha/clipboard.js/archive/master.zip