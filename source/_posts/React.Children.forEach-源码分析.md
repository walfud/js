---
title: React.Children.forEach-源码分析
tag:
  - react
date: 2017/11/15
---

关于为什么会有 `React.Children.forEach` 而不直接用 `this.props.forEach` 的问题, 我在 [React.Children.xxx 的作用](http://js.walfud.com/React.Children.xxx%20%E7%9A%84%E4%BD%9C%E7%94%A8/) 已经说得很明白了. 本文我们要更进一步, 分析一下 `React.Children` 的实现, 从而更好的理解 react 背后的故事. Let's go.

TODO: 

# Refs
[React.Children.xxx 的作用](http://js.walfud.com/React.Children.xxx%20%E7%9A%84%E4%BD%9C%E7%94%A8/)