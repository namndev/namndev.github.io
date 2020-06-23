---
layout: post
title: "Using the OpenSSL library with macOS"
image: https://www.poweradmin.com/blog/wp-content/uploads/2013/10/openssl-logo2.png
tags: [OpenSSL, OSX]
comments: true
---

Working with C++ libraries on a Mac can be a pain and OpenSSL, a very popular one that’s used in many other libraries, led me scrambling around the web and going through different StackOverflow posts, Github issues, and blog posts/forums trying to figure out a way to do it without reinstalling everything.
Usually, you would just install OpenSSL with Homebrew….

```bash
$ brew install openssl
```

And then link it…

```bash
$ brew link --force openssl
Warning: Refusing to link: openssl
Linking keg-only openssl means you may end up linking against the insecure, deprecated system OpenSSL while using the headers from Homebrew's openssl.
Instead, pass the full include/library paths to your compiler e.g.:
   -I~/.development/homebrew/opt/openssl/include -L~/.development/homebrew/opt/openssl/lib
```

But don’t worry! There’s a fix. Homebrew installs OpenSSL but doesn’t link it to `/usr/local/include`, where the compiler looks into during `#include< …>` Thus, you must manually link it instead:

```bash
$ ln -s ~/.development/homebrew/opt/openssl/include/openssl /usr/local/include
```

You also might want to link:

```bash
$ ln -s ~/.development/homebrew/Cellar/openssl@1.1/[version]/bin/openssl /usr/bin/openssl
```

An example of `[version]` is 1.1.1g. And just in case, check whether you compiler looks into `/usr/local/include`. For clang, it is:

```bash
clang -x c -v -E /dev/null
```

And it should output this if clang searches through `/usr/local/include`

```bash
#include <...> search starts here:
/usr/local/include
```

And you should be able to start using OpenSSL! This fix also doesn’t just work with your C++ project but any project(e.g. Ruby project) you are building that uses OpenSSL.
And just a reminder — remember to add `-lssl` and `-lcrypto` flags for your compiler.


_I hope this guide helped solve a headache of yours! I’m pretty new at this and I’d like to get better — any response is welcome :)_