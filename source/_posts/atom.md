---
title: atom
tag:
  - 拒绝无脑推荐
---

拒绝无脑推荐!

想必看到本文的人都是经过纠结后最终选择 atom 的人. 目前看来, 我认为你的选择是明智的, 因为:
* atom 跨平台. 你在 windows / mac / linux 都能用
* atom 拥有大量的插件
* atom 由 GitHub 力推
* atom 开源并且使用 js(nodejs) 编写. 目前看来, js 还是最火的语言没有之一

那么这个号称 '二十一世纪' 的编辑器没有插件一样无法起飞. 以下我介绍常用的快捷键以及几个必备利器.

## 常用
* `Ctrl-n` 新建文件
* `Ctrl-Shift-n` 新建文件夹
* `Ctrl-w` 关闭文件
* `Ctrl-Shift-w` 关闭 atom (慎用! 这个不保存内容!!!)


* `Alt-1` 切换到第 1 个 tab, 同理, 可以使用 2, 3, 4... 切换 tab
* `Ctrl-PageUp` / `Ctrl-PageDown` 快速向左/向右切换 tab


* `Ctrl-t` 快速打开文件
* `Ctrl-f` 当前文件中查找和替换
* `Ctrl-Shift-f` 所有文件中查找和替换
* `Ctrl-r` 在当前文件中查找函数的定义
* `Ctrl-g` 快速跳转到某行


* `Ctrl-Alt-F2` 打标签
* `F2` 跳转到下一个标签
* `Shift-F2` 跳转到上一个标签
* `Ctrl-F2` 列出所有标签


* `Ctrl-Shift-l` 指定当前文件的解析语言
* `Ctrl-Shift-u` 指定当前文件的字符集


* `Ctrl-Alt-[` / `Ctrl-Alt-]` 折叠/展开代码
* `Ctrl-Alt-Shift-[` / `Ctrl-Alt-Shift-]` 全部折叠/展开


* `Ctrl-d` 寻找下一个相同的串并多选 (很实用的列编辑功能)
* `Ctrl-鼠标左键` 多选编辑
* `Ctrl-j` 连接当前行和下一行


* `Ctrl-Shift-p` 打开 command

## Split
这是个内建功能, 而且没有快捷键. 可以通过点击 tab 进行快速分屏:

![](/images/atom/split.png)

## [last-cursor-position](https://atom.io/packages/last-cursor-position)
`Alt--` / `Alt-Shift--` 快速回到上/下次光标的位置.

## [goto-last-edit](https://atom.io/packages/goto-last-edit)
`Ctrl-i` / `Ctrl-Shift-i` 快速回到上/下次编辑过的位置. 这个超级实用

## 找到配对的 ()[]{}
`Ctrl-m` 跳转到对应的 ()[]{}

`Ctrl-Alt-m` 选中匹配项中的所有数据

![](/images/atom/match.png)

## [atom-beautify](https://atom.io/packages/atom-beautify)
`Ctrl-Alt-b` 帮你快速的整理代码.

![](/images/atom/beautify.png)

## [atom-terminal](https://atom.io/packages/atom-terminal)
`Ctrl-Alt-t` 项目根目录下打开终端

`Ctrl-Shift-t` 当前文件根目录下打开终端 (目前版本这个快捷键有 bug, 只有打开过根目录终端后这个快捷键才响应)

## [pigments](https://atom.io/packages/pigments)
帮你识别代码中的颜色

![](/images/atom/pigments.png)

## [minimap](https://atom.io/packages/minimap)
小地图功能也很实用...

![](/images/atom/minimap.png)

## [file-icons](https://atom.io/packages/file-icons)
![](/images/atom/file-icons.png)

# 快速安装
'''
apm install last-cursor-position goto-last-edit atom-beautify atom-terminal pigments minimap file-icons
'''
