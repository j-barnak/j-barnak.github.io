---
layout: post
title:  "Operating Systems Development in C++: Getting Started"
date:   2022-12-22 10:01:30 PM EST
categories: operating-system
---
# Table of Contents
{:.no_toc}
* TOC
{:toc}

# Introduction

For the brave who wish to venture in the strange and arduous world of operating system development, there are neccessary and rewarding challenges you must overcome.
But some of these difficulties are needlessly difficult and only bring frustration. If you decide to develop an operating system in C++, well, that road is certainly
less travelled and wrought with own own difficulties. In this guide, I hope to shed light on the less documented world of C++ operating system development.

# The Goal


< What we will be making >

# Glorious CMake

CMake is definitely a... solution, and you're going to need to trust me on this. Wrestling CMake is hard and debugging CMake errors could be a nightmare. And the documentation is succint to a fault. And 
the syntax is unwieldly. And, well, it's not perfect,  but proper *modern* CMake is quite nice (granted, this may be stockhold syndrome taking its effect on me).

We are going to design our operating system so that it's modular and easy to build onto. This is going to reflect in our CMake file, which is `CMakeLists.txt`. The general gist of it is:

* cmake version
* project name
* creating the executable binary
    * the `a.out` of our operating system, but it'll be a bootable image instead
* the part of the operating system we're working on
    * we will be able to add parts easily of our operating system
    * there will also be a separate `CMakeLists.txt` with each part of the operating you work on
* compiler options
* linker options
    * invoking our linker script
* a file specifying the tools we will be using
    * in other words, our toolchain

That's not so bad, right? The above, as I mentioned, is only a gist of our `CMakeLists.txt` and is really only to prepare you for when we actually dive into the `CMakeLists.txt` file, which will be now! 

```
cmake_minimum_required(VERSION 3.20)
project(NakOS CXX ASM)
```

CMake to know how to call a C++ compiler or assembler

These

# Cross Compiler

< Explain the need for cross compilation >

# Linking

< explain how linking works >

# Linker Script

< Explain why we need a linker script>

# The Bootloader

< How the bootloader works >
