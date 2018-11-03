---
layout: post
title: Emacs, let's take the control back of our workflow
date: 2018-09-20 15:46
comments: true
external-url:
categories: Emacs
---

> We all remember our first program, a simple 'Hello World' or just a calculator that made us all excited. We were all impressed how amazing it's to open a terminal and run or compile it. For that, we used a simple text editor like Notepad++ or others like Eclipse or PyCharm. At that moment, we focused only on the result without worrying about which text editor or IDE to choose.<br/>What if this choice has a big impact on our workflow and determines the quality of the rendering ? If so, what makes a text editor a good one ?

## Overview
While coding, we generally have to do many parallel tasks and sometimes we just get deconcentrated because we have to deal with many tools at the same time. We all wish to have a simple tool that allows us to do all kinds of tasks as quickly as possible.

<a href="https://www.gnu.org/software/emacs/"><em>Emacs</em></a> and <em>Vim</em> are parts of those effective tools that allow you to be more organized and productive, not only in programming but also in everyday life. I personally use Emacs for a couple of years and I am just enjoying using it each time. It's known that learning Emacs could be a bit complicated at first but after using it for a while now, you will define your own workflow.

This article is not a tutorial to learn Emacs but rather to share my own configuration that I got after using emacs for a while now. To learn the Emacs basics, you can follow the <a href='https://www.gnu.org/software/emacs/tour/'>official tutorial</a>.

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
There are a lot of useful packages that we can install and use, we can install them manually by running `M-x package-install` or by just using <a href="https://github.com/jwiegley/use-package">`use-package`</a>, which is a package that allows us to isolate package configuration in our `.emacs` file. At first we only need to install it.

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

## Customization variables
Emacs has many settings which we can change. I prefer isolating the design's customization, like theme, outside of the init file.
```cl
;; save the customization and theme settings in other file
(setq custom-file "~/.emacs.d/custom-settings.el")
(load custom-file t)
```

Keeping backups might be useful generally. To keep them organized we need to set the backup directory.
```cl
(setq backup-directory-alist '(("." . "~/.emacs.d/backups")))
(setq version-control t)
(setq delete-old-versions -1)
```

When we have to write in other languages or use special characters, `utf-8` encoding is very useful. Choosing the right font makes us more comfortable while writing or coding.
```cl
;; encoding system
(set-language-environment "UTF-8")
(set-keyboard-coding-system 'utf-8)

;; change the yes or no answer to y or n
(defalias 'yes-or-no-p 'y-or-n-p)

;; set font
(set-default-font "Monaco-14")

;; change the line spacing for better visualization
(setq-default line-spacing 5)
```

```cl
;; start up buffers
(setq inhibit-splash-screen t
      initial-scratch-message nil
      initial-major-mode 'org-mode)

;; frames configuration
(tool-bar-mode -1)
(display-time-mode 1)
(menu-bar-mode -99)
(scroll-bar-mode -1)

;; marking text
(delete-selection-mode t)
(transient-mark-mode t)
(setq x-select-enable-clipboard t)

;; show ~ at the end of the file for empty lines
(setq-default indicate-empty-lines t)
(when (not indicate-empty-lines)
  (toggle-indicate-empty-lines))
(progn
  (define-fringe-bitmap 'tilde [0 0 0 113 219 142 0 0] nil nil 'center)
  (setcdr (assq 'empty-line fringe-indicator-alist) 'tilde))
```
