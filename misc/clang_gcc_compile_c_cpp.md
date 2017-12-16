---
permalink: /misc/clang_gcc_compile_c_m_cpp
layout: cnblog_post
title:  'clang、gcc编译c、m、cpp'
date:   2017-12-07 06:34:39
categories: misc
---

```
# 使用clang编译m, 注意/usr/bin/cc -> clang
cc –c hello.m
cc hello.o –framework Foundation
./a.out

# 使用clang++编译m, 注意/usr/bin/c++ -> clang++
c++ hello.m -framework Foundation
./a.out

# 使用gcc编译cpp
gcc -l stdc++ hello.cpp
./a.out

# 使用g++编译cpp
g++ hello.cpp
./a.out

# 使用clang编译cpp
clang -l stdc++ hello.cpp
./a.out

# 使用clang++编译cpp
clang++ hello.cpp
./a.out
```