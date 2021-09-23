---
title: CSS 布局
categories:
- 前端
tags:
- CSS
- 手撕
cover: https://tva1.sinaimg.cn/large/00897VxHly8gpqmpouwo5j30sg0hsk3x.jpg
---
本文参考自[几种常见的CSS布局](https://juejin.cn/post/6844903710070407182#heading-12)，感谢。
# 单列布局
### 1. header,content和footer等宽的单列布局
![单列布局-1](https://tva2.sinaimg.cn/large/00897VxHly8gpq40mekbcj31ha0qbwex.jpg)
```html
<!--html-->
<div class="header"></div>
<div class="content"></div>
<div class="footer"></div>
```
```css
/*css*/
.header {
    margin: 0 auto;
    max-width: 960px;
    height: 100px;
    background-color: blue;
}
.content {
    margin: 0 auto;
    max-width: 960px;
    height: 545px;
    background-color: green; 
}
.footer {
    margin: 0 auto;
    max-width: 960px;
    height: 100px;
    background-color: yellow;
}
```
### 2. header与footer等宽，content略窄的单列布局
![单列布局-2](https://tva3.sinaimg.cn/large/00897VxHly8gpq40y23iej31hc0q8q3e.jpg)
```html
<!--html-->
<div class="header">
    <div class="nav"></div>
</div>
<div class="content"></div>
<div class="footer"></div>
```
```css
/*css*/
.header {
    margin: 0 auto;
    max-width: 960px;
    height: 100px;
    background-color: blue;
}
.nav {
    margin: 0 auto;
    max-width: 800px;
    height: 50px;
    background-color: darkgray;
}
.content {
    margin: 0 auto;
    max-width: 800px;
    height: 545px;
    background-color: green;
}
.footer {
    margin: 0 auto;
    max-width: 960px;
    height: 100px;
    background-color: aqua;
}
```
# 两列布局
### 1. table布局
![table布局-1](https://tva4.sinaimg.cn/large/00897VxHly8gpq4unt8daj30s70hndfu.jpg)
```html
<!--html-->
<table>
    <tr>
        <td class="left"></td>
        <td class="right"></td>
    </tr>
</table>
```
```css
/*css*/
table {
    width: 800px;
    height: 500px;
    border-collapse: collapse; /*合并表格边框**/
}
.left {
    background-color: brown;
}
.right {
    background-color: burlywood;
}
```
### 2. table class布局
布局效果图和上方的table布局一样，相当于用div模拟出了一个table。
```html
<!--html-->
<div class="table">
    <div class="table_row">
        <div class="left table_cell"></div>
        <div class="right table_cell"></div>
    </div>
</div>
```
```css
/*css*/
.table {
    display: table;
    width: 800px;
    height: 500px;
}
.table_row {
    display: table-row;
}
.table_cell {
    display: table-cell;
    vertical-align: middle;
}
.left {
    background-color: brown;
}
.right {
    background-color: burlywood;
}
```
# 两列自适应布局
两列自适应布局是指一列由内容撑开，另一列撑满剩余宽度的布局方式。
![两列自适应布局](https://tva2.sinaimg.cn/large/00897VxHly8gpq9640wuqj31gz074glk.jpg)
### 1. float + BFC(overflow: hidden)
```html
<!--html-->
<div class="left">固定</div>
<div class="right">自适应</div>
```
```css
/*css*/
.left {
    float: left;
    width: 200px;
    height: 300px;
    background-color: yellow;
}
.right {
    overflow: hidden;
    height: 300px;
    background-color: green;
}
```
### 2. absolute + margin 方式
```html
<!--html-->
<div class="container">
    <div class="sidebar">固定</div>
    <div class="main">自适应</div>
</div>
```
```css
/*css*/
.container {
    position: relative;
}
.sidebar {
    position: absolute;
    top: 0;
    left: 0;
    height: 300px;
    width: 200px;
    background-color: yellow;
}
.main {
    margin-left: 200px;
    height: 300px;
    background-color: green;
}
```
### 3. float + margin 方式
```html
<!--html-->
<div class="sidebar">固定</div>
<div class="main">自适应</div>
```
```css
/*css*/
.sidebar {
    float: left;
    height: 200px;
    width: 200px;
    background-color: yellow;
}
.main {
    margin-left: 200px;
    height: 200px;
    background-color: green;
}
```
### 4. float + 负margin 方式
```html
<!--html-->
<div class="wrap">
    <div class="main">自适应</div>
</div>
<div class="sidebar">固定</div>
```
```css
/*css*/
.wrap {
    float: left;
    width: 100%;
}
.main {
    margin-left: 200px;
    height: 300px;
    background-color: green;
}
.sidebar {
    float: left;
    margin-left: -100%;
    height: 300px;
    width: 200px;
    background-color: yellow;
}
```
### 5. flex 方式
```html
<!--html-->
<div class="parent">
    <div class="left">固定</div>
    <div class="right">自适应</div>
</div>
```
```css
/*css*/
.parent {
    display: flex;
}
.left {
    width: 200px;
    height: 300px;                /*容器的高度是被left模块撑起来的*/
    background-color: yellow;
}
.right {
    flex: auto;
    background-color: green;
}
```
### 6. grid 方式
```html
<!--html-->
<div class="parent">
    <div class="left">固定</div>
    <div class="right">自适应</div>
</div>
```
```css
/*css*/
.parent {
    display: grid;
    grid-template-columns: 200px auto;
    grid-template-rows: 300px;
}
.left {
    background-color: yellow;
}
.right {
    background-color: green;
}
```
# 三栏布局
![三栏布局](https://tva4.sinaimg.cn/large/00897VxHly8gpqdf307ttj31gz0anq2y.jpg)
- `特征`：中间列自适应宽度，旁边两侧固定宽度
- `种类`：圣杯布局和双飞翼布局
- `差异`：圣杯布局通过父容器的内边距来实现各列的间隙；双飞翼布局通过新建的div的外边距隔离各列
### 1. 圣杯布局
```html
<!--html-->
<article class="parent">
    <div class="center"></div>
    <div class="left"></div>
    <div class="right"></div>
</article>
```
```css
/*css*/
.parent {
    margin: 0 auto;
    padding: 0 100px;
}
.left, .center, .right {
    float: left;
    height: 300px;
}
.left {
    position: relative;
    width: 100px;
    left: -100px;
    margin-left: -100%;
    background-color: green;
}
.center {
    width: 100%;
    background-color: yellow;
}
.right {
    position: relative;
    width: 100px;
    right: -100px;
    margin-left: -100px;
    background-color: blue;
}
```
### 2. 双飞翼布局
```html
<!--html-->
<div class="container">
    <div class="content"></div>
</div>
<div class="left"></div>
<div class="right"></div>
```
```css
/*css*/
.left, .container, .right {
    float: left;
    height: 300px;
}
.container {
    width: 100%;
    background-color: yellow;
}
.content {
    margin: 0 100px;
}
.left {
    width: 100px;
    margin-left: -100%;
    background-color: green;
}
.right {
    width: 100px;
    margin-left: -100px;
    background-color: blue;
}
```
### 3. flex 实现三栏布局
```html
<!--html-->
<div class="container">
    <div class="center"></div>
    <div class="left"></div>
    <div class="right"></div>
</div>
```
```css
/*css*/
.container {
    display: flex;
    height: 300px;
}
.left {
    order: -1;                   /*使left能排在最前面*/
    width: 200px;
    background-color: green;
}
.center {
    flex: auto;                  /*自动填充剩余空间*/
    background-color: yellow;
}
.right {
    width: 200px;
    background-color: blue;
}
```