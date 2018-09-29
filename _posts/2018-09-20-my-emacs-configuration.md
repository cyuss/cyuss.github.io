---
layout: post
title: My Emacs configuration
date: 2018-09-20 15:46
comments: true
external-url:
categories: Emacs
---

> We all remember our first program, a simple 'Hello World' or just a calculator that made us all excited. We were all impressed how amazing it's to open a terminal and run or compile our program. For that, we used a simple text editor like Notepad++ or even an advanced ones like Eclipse or PyCharm. At that moment, we focused only on the result without worrying about which editor to choose.<br/>What if this choice has a big impact on our workflow and determines the quality of the rendering ? If so, what makes a text editor a good one ?

## Overview
While coding, we generally have to do many parallel tasks and sometimes we just get deconcentrated because we have to deal with many tools at the same time. We all wish to have a simple tool that allows us to do all kind of tasks as quickly as possible.

<a href="https://www.gnu.org/software/emacs/"><em>Emacs</em></a> and <em>Vim</em> are one of those effective tools that allow you to get more organized and be more productive, not only in programming but also in everyday life. I personally use Emacs for a couple of years and I am just enjoying using it each time. It's known that learning Emacs could be a bit complicated at the first but after using it for a while, you will define your own workflow.

This article is not a tutorial to learn Emacs but rather to share my own configuration that I got after using emacs for a while now. 

## Download & installation
Emacs is a multiplatform software, it runs on several operating systems. To download and install it, you can follow the instructions in the official home <a href="https://www.gnu.org/software/emacs/download.html">page</a>.

After the installation, we will start the configuration. For that, Emacs has a configuration file which is `~/.emacs` that we can find or create in the home directory.

## Initialization
First of all, we need to initialize Emacs and connect it to the different repositories that will allow us to download any package later.

```cl
(load "package")
(load "cl")
(package-initialize)

(unless (assoc-default "marmalade" package-archives)
  (add-to-list 'package-archives '("marmalade" . "https://marmalade-repo.org/packages/") t))
(unless (assoc-default "melpa" package-archives)
  (add-to-list 'package-archives '("melpa" . "http://melpa.org/packages/") t))
(unless (assoc-default "org" package-archives)
  (add-to-list 'package-archives '("org" . "http://orgmode.org/elpa/") t))
```

## Installing packages
There are a lot of useful packages that we can install and use, we can install them manually by running `M-x package-install` or by just using <a href="https://github.com/jwiegley/use-package">`use-package`</a>, which is a package that allows us to isolate package configuration in our `.emacs` file. So, at first we only need to install it.

```cl
(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))
;; enable use-package
(eval-when-compile
  (require 'use-package))
(require 'bind-key)
```

Let's try installing and configuring `diminish` package. It allows us to hide minor-modes in the mode line displays.

```cl
(use-package diminish
  :ensure t ;; keyword that causes the package to be installed automatically
  :diminish eldoc-mode)
```

## Customize

```cl
(setq custom-file "~/.emacs.d/custom-settings.el")
(load custom-file t)
```
