---
title: JavaScript 学习笔记
tags:
  - JavaScript
  - JS
categories:
  - JavaScript
keywords:
  - JavaScript
  - JS
abbrlink: cedc43fd
date: 2017-08-08 20:54:56
---
## 前言

基本理论了解，只是为了记住JS众多的函数

<!--more-->

## 属性

* `display=block` 显示  
* `display=none` 隐藏  
* `length` 数组个数
* `checked` true 选中  
* `innerHTML` (inner里边的)HTML  
* `style` 获取行间样式  
* `oDiv.currentStyle.width` 获取非行间样式(只兼容IE)
* `getComputedStyle(oDiv,null).width` 获取非行间样式（chrome FF）获取计算后的样式 兼容IE9+
* `offsetLeft` 获取元素的左边距
* `offsetTop` 获取元素顶部边距
* `offsetWidth` 获取元素的宽
* `offsetHeight` 获取元素的高
* `childNodes` 查看该节点下有几个子节点(数组) 空白也算一个节点
* `nodeType` 获取节点类型
* `children` 指包括元素子节点，不包括文本子节点(数组)-（兼容)
* `parentNode` 查找某一个元素的父节点
* `offsetParent` 查找某个元素的父级定位元素(结合css相对绝对定位)
* `firstElementChild` 某元素第一个子节点
* `lastElementChild` 某元素最后一个子节点
* `nextElementSibling` 下一个兄弟节点
* `previousElementSibling` 上一个兄弟节点
* `className` 获取 class 名
*


## 事件

* `onclick` 当按钮被点击时  
* `onmouseover` 当鼠标移入时  
* `onmouseout` 当鼠标移出时  
*

## 函数

* `arguments` 可变餐，不定参（是一个参数的数组）

### 系统方法  

* `alert()` 弹框  
* `typeof` typeof a 获取数据类型  
* `parseInt()` 转为整型  
* `parseFloat()` 转为浮点型  
* `setInterval(fn,ms)` 每隔多少毫秒执行一个函数  
* `setTimeout(fn,ms)` 隔多少毫秒执行这个函数  
* `clearInterval(var setInterval的返回值)` 关闭定时器
* `clearTimeout(var setTimeout的返回值)` 关闭定时器

### 数组操作方法  

* `push(元素)` 从尾部添加  
* `unshift(元素)` 从头部添加
* `pop()` 从尾部删除  
* `shift()` 从头部删除
* `splice(起点，长度)` 从 *起点* 开始删 *长度* 个
* `splice(起点，长度=0，元素...)` 从 *起点* 开始删 *0* 个 插入 *元素...* 。
* `concat(数组)` 连接两个数组 a.concat(b)
* `join(连接符)` 数组元素用连接符连接起来
* `sort(比较函数)` 排序

### 字符串操作方法

* `charAt(0)` 可以获取字符串上某一位的值 全兼容 `str[0]` 不兼容低版本
* `search(字符串)` 搜索并返回字符串出现的位置，没找到返回 -1（模糊搜索）
* `split(分割符)` 字符串分割，返回数组

## 对象

### Date 对象

* `getHours()` 获得小时
* `getMinutes()` 获得分钟
* `getSeconds()` 获得秒数
* `getFullYear()` 获得年份
* `getMonth()` 获得月份（从 0 开始)
* `getDate()` 获得日
* `getDay()` 获得星期(0 是周日)

## DOM

* `document.getElementById('')` 获取Id元素
* `document.getElementsByTagName('')` 获取一组元素   
* `document.setAttribute('属性名','值')` DOM设置属性值
* `document.getAttribute('属性名')` 获取属性值
* `document.removeAttribute('属性名')` 删除属性
* `document.createElement(标签名)` 创建元素(标签)
* `父级.appendChild(子节点)` 先删除原来父级上的元素，再给父级添加子节点（之后），如果原父级没有，直接添加。（排序）
* `父级.insertBefore(子节点,在谁之前)` 给父级添加子节点（之前）
* `父级.removeChild(子节点)` 从父级删除子节点
*

## BOM

* `window.onload` 页面加载完成时  
