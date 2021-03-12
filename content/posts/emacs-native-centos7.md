+++
title = "Emacs native-comp on CentOS 7"
date = 2020-05-19
tags = ["emacs"]
draft = false
+++

The GNU Emacs `feature/native-comp` branch has been under development
for some time now. The performance enhancements from the natively
compiled Emacs Lisp code are exciting. Notably, I've been seeing nice
speed-ups for [Helm](https://emacs-helm.github.io/helm/) completions and a smoother [lsp-ui](https://emacs-lsp.github.io/lsp-ui/) experience.

Andrea Corallo is developing this new feature and updates/descriptions
of the work can be tracked/found [on his website](http://akrl.sdf.org/gccemacs.html). I've been building
the branch on my CentOS 7 machine for a few weeks now, and I thought
I'd walk through the process.


## Building Emacs using `--with-native-compilation` {#building-emacs-using-with-native-compilation}

****Update March 2021****: The `configure` option was originally
`--with-nativecomp`, but it has changed to `--with-native-compilation`.

We need to install `libgccjit`. Unfortunately CentOS 7 shipped with a
pretty old GCC release (the 4.8 series). Fortunately Red Hat ships
modern GCC builds with a number of `devtoolset` RPMs. We can install
`libgccjit` (and the necessary development headers) from the GCC 9
series via:

```nil
# yum install devtoolset-9-gcc devtoolset-9-libgccjit-devel
```

We'll build Emacs from source after checking out the feature
branch:

```nil
$ git clone https://git.savannah.gnu.org/git/emacs.git
$ cd emacs
$ git fetch --all
$ git checkout -b native-comp origin/feature/native-comp
```

Now we'll build Emacs after enabling `devtoolset-9`. We ensure that
`pkg-config` will search in `/usr/lib64/pkgconfig` for installed
packages, such as `gnutls` or `libjansson` installed with `yum` (this
is necessary because we are installing with GCC 9 from `devtoolset-9`
and not the default `/usr/bin/gcc` compiler). (Since we're playing
with an experimental feature, I'm going to assume that you've built
Emacs from source before and that you can handle all other desirable
`configure` options).

```nil
$ source /opt/rh/devtoolset-9/enable
$ ./autogen.sh
$ PKG_CONFIG_PATH=/usr/lib64/pkgconfig ./configure --with-native-compilation
$ make -j6 NATIVE_FULL_AOT=1
```

Notice the use of `NATIVE_FULL_AOT=1`. This will ask Emacs to compile
<span class="underline">all</span> builtin Emacs Lisp code natively (the AOT stands for
ahead-of-time). If this option is not used, only bare minimum will be
natively compiled, and lot of the shipped Emacs Lisp packages will be
regular byte compiled. If you enable deferred compilation (keep
reading), those package will be natively compiled on the fly (the
first time they are loaded, that's the just-in-time compilation). I
prefer to natively compile <span class="underline">all</span> of Emacs when I build it, that way
Emacs will only just-in-time compile my third-party packages.

Once Emacs is compiled we can run it with `src/emacs` (you can set an
install prefix, but this is an experimental feature, so I only run
this executable from the development repository and keep a `master`
branch build installed somewhere in my `PATH`). The remainder of this
post is not specifically related to CentOS 7, but it's still useful.


## Deferred and asynchronous compilation {#deferred-and-asynchronous-compilation}

Before we run Emacs we can add a few lines to the top of our `init.el`
file to steer deferred/async compilation. When running Emacs with the
`native-compile-async` symbol defined, we ask if we want to run the
deferred async compilation. If yes, set the number of jobs that can
run in the background. There are a few other variables that control
native compilation. I recommend giving the `comp-` customization group
a quick read.

```emacs-lisp
;; helper boolean I use here and later in my init.el
(defconst dd/using-native-comp-p (fboundp 'native-comp-available-p))
(when dd/using-native-comp-p
  (setq comp-async-query-on-exit t)
  (setq comp-async-jobs-number 4)
  (setq comp-async-report-warnings-errors nil)
  (setq comp-deferred-compilation t))
```

Emacs will asynchronously natively compile all `.elc` files that it
loads. So if your `init.el` file loads a lot of packages, prepare for
Emacs to spend a bit of time compiling. Fortunately you can still use
Emacs while that is happening in the background.
