---
id: introduction
title: Trident简介
---

## Trident的目标
一套可快速开发支持商业运营App的框架。

## Trident的开发背景
Trident由微众银行App团队开发，开发过程中我们通过调研其他一些RN的开发框架发现发现大部分适合比较小的项目，或者是用到后期要真实运营的适合还需要补充很多运营相关的基础能力。
要开发真正商业运营的App，需要很多打磨，没有办法做到快速开发高质量产品，以此为出发点我们开发了Trident，希望在开发、运营、测试几方面提供更好的基础开发设施。

## Trident给你提供了什么
不纠结和避免选择困难症
最实用而非最时尚的技术框架。

## 基础
在使用Trident之前，我们希望你能了解一些基础知识。
从我们开发经验来说一个App要做的主要事情包括如下内容，

// TODO 思维导图

以此为基础，Trident始终围绕上述的这些点去尽可能简化App的开发。

- 路由是一个应用必不可少的部分，Alita提供了简单易用的路由组件@areslabs/router。并且只支持转化使用了这个路由的RN应用
- 数据状态