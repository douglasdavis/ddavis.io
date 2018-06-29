---
layout: post
title: "Yet Another Emacs C++ IDE"
date: 2018-06-27
---

Lately I've seen a lot of posts on the
[Emacs](https://old.reddit.com/r/emacs) and
[C++](https://old.reddit.com/cpp) subreddits related to Emacs as a C++
IDE. If you give the topic a quick googling you'll see a lot of posts
and tutorials which walk through using
[cquery](https://github.com/cquery-project/cquery),
[lsp-mode](https://github.com/emacs-lsp/lsp-mode),
[rtags](https://github.com/Andersbakken/rtags),
[irony](https://github.com/Sarcasm/irony-mode),
[company](http://company-mode.github.io/),
[ycmd](https://github.com/abingham/emacs-ycmd), etc. (obviously there
are a number of options out there and multiple blog posts and
tutorials for each). I've personally tried using cquery and rtags
(both
[libclang](https://github.com/llvm-mirror/clang/tree/master/tools/libclang)
based) in combination with company-mode. My configuration was hacked
together in an unorganized way and I was never completely satisfied
with the experience.

I've recently landed on a new setup using lsp-mode and
[lsp-clangd](https://github.com/emacs-lsp/lsp-clangd). As is clear
from the package name, this method takes advantage of (the rapidly in
development) LLVM tool
[clangd](https://github.com/llvm-mirror/clang-tools-extra/tree/master/clangd).
