+++
title = "How to use ox-hugo"
author = ["JinZhongjia"]
date = 2024-08-11
layout = "blog"
lastmod = 2024-08-11T21:55:57+08:00
slug = ""
draft = false
+++

[ox-hugo](https://ox-hugo.scripter.co/) is a emacs plugin for hugo which can help us to use org in hugo.

It will convert `.org` file to `.md` file.

<!--more-->


## Install {#install}

We can install it easily, if you use emacs plugin manager `elpaca`, just use this:

```emacs-lisp
(use-package ox-hugo
  :ensure t
  :defer t
  :custom
  (org-hugo-auto-set-lastmod t "auto update latest time")
  )
```

`use-package` will automatically install it, `:defer t` will make it lazy load.


## Config {#config}

For configuration of `ox-hugo`, we need to understand several basic variables.

-
