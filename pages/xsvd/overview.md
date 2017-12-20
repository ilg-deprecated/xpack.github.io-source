---
layout: page
lang: en
permalink: /xsvd/
title: xsvd
author: Liviu Ionescu

date: 2017-10-11 14:25:00 +0300

---

## Overview 

The **XSVD** format is inspired from ARM [CMSIS SVD](http://www.keil.com/cmsis/svd), which is based on XML and was influenced by [IP-XACT](https://en.wikipedia.org/wiki/IP-XACT).

The XSVD content is generally similar to the SVD content, but it has a better structure, using a hierarchy of objects with properties. As such, the natural format to represent it is JSON, which is simpler and easier to parse than XML.

To help the migration from CMSIS Packs to xPacks, an automated tool was written to convert ARM SVD to XSVD.

## Purpose

Contrary to the wider scope of IP-XACT, the XSVD format was intentionally kept as simple as possible, since it is currently intended only for:

* allowing debuggers to display memory mapped peripherals, including separate register bit-fields
* supporting the automated generation of device header files
* supporting the automated generation of source files implementing the peripherals in emulators (like QEMU).

## File formats

* [xsvd.json]({{ site.baseurl }}/xsvd/files/xsvd-json/)
