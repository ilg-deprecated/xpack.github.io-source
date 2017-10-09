---
layout: page
lang: en
permalink: /xmake/cli/xmake-build/
title: xmake build - Build one or more configurations
author: Liviu Ionescu

date: 2017-10-09 13:06:00 +0300

---

Generate the build files and build.

## Synopsis

```
xmake build [--target <name>]* [--profile <name>]* [--toolchain <name>]* [-- <args>]
```

Aliases:
- `b`
- `bild`

## Description

This command expects an `xmake.json` file in the CWD, to define the 
build configurations.

A **build configuration** is a triplet (target, profile, toolchain).

For each configuration, `xmake build` creates a subfolder in the CWD, 
named `build/<name>-<target>-<toolchain>-<profile>`.

If multiple names are defined for target/toolchain/profile, a 
matrix of configurations is constructed.

All names must be letters, hyphens, or digits. When used to 
create paths, all letters are converted to lower case.

After generating the build folders, the native builder (like `make`) 
is invoked with the extra arguments.

## Examples

```
$ cd xyz-xpack.git
$ xmake build -- clean all
```

When executed, this command creates sub-folders like `darwin-debug-clang` and 
`darwin-release-clang` and `make` is invoked in each folder
to run the actual build. 

