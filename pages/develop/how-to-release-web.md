---
layout: page
lang: en
permalink: /develop/how-to-release-web/
title: How to release the project web?
author: Liviu Ionescu

date: 2016-07-11 20:50:00 +0300

---

## Check the jekyll site

In Finder,

* double click the `jekyll-serve.mac.command`
* double click the `localhost-4002.webloc`

Verify the changed pages.

## Add post to announce new release

* in `_posts/releases/`
* add a new post named file like `2016-11-27-xpm-v6-3-10-released.md`

## Build the jekyll site

In Finder,

* double click the `jekyll-build.mac.command`

## Publish site

With GitHub Desktop,

* commit `xpack.github.io`
* click the **Sync** button
