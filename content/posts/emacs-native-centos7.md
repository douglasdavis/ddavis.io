+++
title = "Emacs native-comp on CentOS 7"
date = 2020-05-19
tags = ["emacs"]
draft = false
+++

**As of March 2021 this post is a bit outdated.** I've added [a new
section](#march-2021-update) (at the end) outlining some updates.

The GNU Emacs `feature/native-comp` branch has been under development
for some time now. The performance enhancements from the natively
compiled Emacs Lisp code are exciting. Notably, I've been seeing nice
speed-ups for [Helm](https://emacs-helm.github.io/helm/) completions and a smoother [lsp-ui](https://emacs-lsp.github.io/lsp-ui/) experience.

Andrea Corallo is developing this new feature and updates/descriptions
of the work can be tracked/found [on his website](http://akrl.sdf.org/gccemacs.html). I've been building
the branch on my CentOS 7 machine for a few weeks now, and I thought
I'd walk through the process.


## Building Emacs using `--with-nativecomp` {#building-emacs-using-with-nativecomp}

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
and not the default `/usr/bin/gcc` compiler). We use
`NATIVE_FAST_BOOT` to shorten the compilation process; with this
option only Emacs Lisp code necessary for a base Emacs installation
will be natively compiled. We're deferring the compilation of other
Emacs Lisp code. (Since we're playing with an experimental feature,
I'm going to assume that you've built Emacs from source before and
that you can handle all other desirable `configure` options). (UPDATE
June 2020: as of mid June 2020 the compilation time has been
drastically improved, making the `NATIVE_FAST_BOOT` option not as
useful as it was before that time).

```nil
$ source scl_source enable devtoolset-9
$ ./autogen.sh
$ PKG_CONFIG_PATH=/usr/lib64/pkgconfig ./configure --with-nativecomp
$ make -j6 # use NATIVE_FAST_BOOT=1 if desired
```

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
run in the background (one can also define a blacklist. The blacklist
is useful for avoiding compiling Emacs Lisp code that we don't often
use, see `C-h v comp-deferred-compilation-black-list` for more, it
looks for regex matches).

```emacs-lisp
;; for native-comp branch
(when (fboundp 'native-compile-async)
  (if (yes-or-no-p "async compile?")
      (setq comp-async-jobs-number 4 ;; not using all cores
            comp-deferred-compilation t
            comp-deferred-compilation-black-list '())
    (setq comp-deferred-compilation nil)))
```

Emacs will asynchronously natively compile all `.elc` files that it
loads. So if your `init.el` file loads a lot of packages, prepare for
Emacs to spend a bit of time compiling. Fortunately you can still use
Emacs while that is happening in the background.


## March 2021 update {#march-2021-update}

The `native-comp` branch is quite close to being merged in to the main
GNU Emacs development branch. When version 28 is released (no schedule
for that yet) it is expected that `native-comp` will be an
experimental feature. The `configure` option name has changed, and a
few other customizations have changed. Here's a brief run-down of how
I'm currently building native-compilation supporting GNU Emacs on
CentOS 7:

```nil
$ source /opt/rh/devtoolset-9/enable
$ ./autogen.sh && \
    PKG_CONFIG_PATH=/usr/lib64/pkgconfig \
    CFLAGS="-O3 -mtune=native" \
    ./configure \
    --with-native-compilation \
    --with-modules \
    # ... other options
$ make -j6 NATIVE_FULL_AOT=1
```

The `NATIVE_FULL_AOT` option does full ahead-of-time compilation of
the Emacs Lisp packages shipped with GNU Emacs. And in my `init.el`:

```emacs-lisp
;; helper boolean I use here and later in my init.el
(defconst dd/using-native-comp-p (fboundp 'native-comp-available-p))
(when dd/using-native-comp-p
  (setq comp-async-query-on-exit t)
  (setq comp-async-jobs-number 4)
  (setq comp-async-report-warnings-errors nil)
  (setq comp-deferred-compilation t))
```
