---
layout: post
title:  "Eclipse插件plugin.xml只能增加Generic扩展点"
date:   2019-04-23 22:21:01 +0800
categories: RCP开发
tag: RCP开发
---


* content
{:toc}


Eclipse RCP开发，扩展点处只能创建Generic扩展点			{#this.title}
====================================
Eclipse RCP开发，扩展点处的"new"上下文只能创建Generic节点，这主要是由于**没有安装RCP开发环境的扩展点的schame，集成环境的可视化插件不能分析出正确的语法。**

1. 使用菜单栏“help”下的“install new software”；
2. 在弹出“install”对话框里，选择官网地址，我这里是http://download.eclipse.org/technology/epp/packages/oxygen/，即[官网地址](http://download.eclipse.org/technology/epp/packages/oxygen/)；
3. 在下边的提示为“type filter text”的编辑框里输入RCP，然后就会搜索出RCP的相关插件；
4. 点击重新安装下，然后重启Eclipse。