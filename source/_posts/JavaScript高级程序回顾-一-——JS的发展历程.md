---
title: JavaScript高级程序回顾(一)——JavaScript简介
date: 2018-01-08 12:38:40
tags:
---

### JavaScript的诞生
　　在早期人们普遍使用电话拨号上网的时代，上网速度仅为 kb/s 级别。想象一下，你辛辛苦苦填完了一个表单，点击“提交”按钮后等待着奇迹的来临，然而...30秒以后，服务器返回消息说你有一个必填项没填好，是不是有一种拿小拳拳锤坏电脑的冲动！？有这种冲动的当然不止你一个人，当时就职于 Netscape 公司的 Brendan Eich 着手开发一门语言用以解决这个问题， 这就是后来的 JavaScript。

### 什么是标准？
　　然鹅，伴随着 Netscape Navigator2 的正式发布，其内置的 JavaScript1.0 取得了很大成功。微软会不眼红吗？所以在后来的 IE 浏览器中就看到了功能类似的 JScript。至此，意味着 JavaScript 的有了不同的版本，但是却没有标准规定 JavaScript 的语法和特性，两个公司之间的战争，最大的受害者是 Web 工作者，随着业界担心的加剧，JavaScript 的标准化问题被提上了日程。
　　接下来，欧洲计算机制造商协会(ECMA) 经过数月的努力完成了 ECMA-262 —— 定义一种名为 ECMAScript 的新脚本语言的标准，第二年，ISO 也采用了 ECMAScript 作为标准。自此以后，浏览器开发商致力于在 ECMAScript 的基础上实现 JavaScript。

### JavaScript实现
　　虽然 JavaScript 和 ECMAScript 通常被人们用来表达相同的含义，但是 JavaScript 的含义却比 ECMA-262 中规定的要多得多。一个完整的 JavaScript 实现由下面三个部分组成：
1. 核心(ECMAScript)
ECMA-262 第一版本质上与 JavaScript1.1 相同，只不过删除了所有针对浏览器的代码并作了一些小的改动。所以，ECMAScript 与 Web 浏览器并没有依赖关系。我们常见的 Web 浏览器只是 ECMAScript 实现的可能宿主环境之一，现在流行的 Node(一种服务端 JavaScript 平台)也是其宿主环境之一。
2. 文档对象模型(DOM)
本质上，DOM 是针对 HTML 的应用程序编程接口(API)。DOM 把整个页面映射为一个多层节点结构，也就是我们口中常说的“树结构”。借助 DOM 提供的 API，开发人员可以轻松自如地删除、添加、替换或修改任何节点。
3. 浏览器对象模型(BOM)
开发人员可以使用 BOM 控制浏览器显示的页面以外的部分，也就是跟浏览器本身相关的部分。
