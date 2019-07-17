title: soong
categories: Android
tags: [Android]
---

# Table of Contents
- [1. Android.bp file format](#section1)


Soong用来替换make-based的编译系统。

<a name="section1"></a>
# 1. Android.bp file format

## Modules
android.bp中的module通过一个module type开始，后面是一系列name: value的属性。

	cc_binary {
	    name: "gzip",
	    srcs: ["src/test/minigzip.c"],
	    shared_libs: ["libz"],
	    stl: "none",
	}

