---
layout: main
lang: en
permalink: /
title: The xPack Project
author: Liviu Ionescu

date: 2017-10-01 15:34:00 +0300

---

## Mission statement

Provide a set of tools to manage, configure and build complex, package based, multi-target (multi-architecture, multi-board, multi-toolchain), intended to **enhance code sharing** and **reusing** during the **C/C++** libraries and applications life cycle.

## xPacks overview

**xPacks** are general purpose software C/C++ packages, much the same as the highly successful [npm modules](https://docs.npmjs.com/getting-started/what-is-npm) in the [Node.js](https://nodejs.org/en/)/JavaScript ecosystem.

## xPack tools

The entire xPack project is open source and publicly available on [GitHub](https://github.com/xpack).

The main xPack tools are:

* [xpm]({{ site.baseurl }}/xpm/) - the package manager ([GitHub](https://github.com/xpack/xpm-js), [npm](https://www.npmjs.com/package/xpm))
* [xmake]({{ site.baseurl }}/xmake/) - the project builder ([GitHub](https://github.com/xpack/xmake-js), [npm](https://www.npmjs.com/package/xmake))
* [xcdl]({{ site.baseurl }}/xcdl/) - the component & configuration framework

The tools are more or less independent, for example both `xpm` and `xmake` can be run without each other, but together may benefit from shared configurations.

## License

Unless otherwise mentioned, all **xPack** tools are provided **free of charge** under the terms of the [MIT License](https://opensource.org/licenses/MIT).
