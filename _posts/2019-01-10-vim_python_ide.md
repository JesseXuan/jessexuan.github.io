---
layout: post
title:  打造python的vim编辑器
date:   2019-01-10 10:48:00 +0800
categories: Documents
tag: 教程
---

* content
{:toc}


设置步骤
====================================
+ 下载插件管理器
```bash
#git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
+ 编辑配置.vimrc，添加需要的插件
+ 运行vim，在普通模式下输入PluginInstall进行插件安装
+ 手动安装自动补全插件：pydiction
  1. cd ~/.vim/bundle, git clone https://github.com/rkulla/pydiction.git
  2. cp -r ~/.vim/bundle/pydiction/after/ ~/.vim
  3. mkdir -p ~/.vim/tools/pydiction, cp pydiction/complete-dict ~/.vim/tools/pydiction
