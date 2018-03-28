---
title: git commit
date: 2017-07-08 22:11:37
tags: git
---
![git](https://user-images.githubusercontent.com/12566627/30576565-6191d128-9cce-11e7-8c82-719278b12f2f.png)
# 前言
  平时总感觉commit message这块，很容易被忽视掉，但提交信息应该清晰明了，说明本次提交的目的，和具体的改动。
## Git commit日志基本规范
Commit message 都包括三个部分：Header，Body 和 Footer
<!--more-->
``` bash
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```
其中Header 是必需的（包括三个字段type、scope、subject），每次提交不超过50个字符
#### 1、type（代表某次提交的类型说明）
* feat：新功能（feature）
* fix: 修复bug
* chore：构建过程或辅助工具的变动
* docs: 修改文档，比如README, CHANGELOG, CONTRIBUTE等
* style: 修改空格、格式缩进等，不改变代码逻辑
* refactor：重构（即不是新增功能，也不是修改bug的代码变动）
* perf: 优化相关，比如提升性能、体验
* revert: 回滚到上一个版本
* test：增加测试

#### 2、scope（影响的范围）
* scope用括号来说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

#### 3、 subject（commit 的简短描述）
* 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
* 第一个字母小写
* 结尾不加句号（.）
<div class="tip">
  这里可以参考angular git commit规范 [这里](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)
<br />
  如果提交massage错误需要修改参考[这里](http://stackoverflow.com/questions/179123/how-to-modify-existing-unpushed-commits)
</div>