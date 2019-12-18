---
layout: post
title: Build SPEC2006 using LLVM's cmake infrastructure
tags:
- LLVM
- CMake
- SPEC CPU 2006 Benchmark
date: 2017-08-21
---
Instructions to build Spec2006 using LLVM's Cmake infrastructure.


```shell
  # Setup the environment.
  export LLVM_SRC=<Root of LLVM source code tree>
  export LLVM_BLD=<Root of LLVM build  tree>
  export SPEC_SRC=<Root of SPEC 2006 source code tree>
  TESTSUITE_BUILD_DIR=<Build dir of test-suite>

  # Make SPEC source available to LLVM build system.
  mkdir $LLVM_SRC/projects/test-suite/test-suite-externals
  ln -s $SPEC_SRC $LLVM_SRC/projects/test-suite/test-suite-externals/speccpu2006

  mkdir $TESTSUITE_BUILD_DIR && cd $TESTSUITE_BUILD_DIR
  # Configure
  cmake $LLVM_SRC/projects/test-suite -DCMAKE_C_COMPILER=<c compiler> -DCMAKE_CXX_COMPILER=<c++ compiler>
  # Build the binaries.
  cd External/SPEC/CINT2006/
  make -j 8
  # Run
  lit -v -j 8 . -o results.json
```
