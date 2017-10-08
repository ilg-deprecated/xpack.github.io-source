---
layout: post
lang: en
title: The xPack new web site
author: Liviu Ionescu

date: 2017-10-01 15:26:00 +0300

categories:
  - news
  - web

---

After several re-branding steps, the xPack project (formerly XCDL), available from [GitHub](https://github.com/xpack), has a new web site.

### Repositories

The current project repositories are:

* [xpack/xpm-js](https://github.com/xpack/xpm-js) - xpm
* [xpack/xmake-js](https://github.com/xpack/xmake-js) - xmake

## The web has a new look

Since [GitHub Pages](https://pages.github.com) is the GitHub solution for providing documentation sites, and this service makes the sites available in the `github.io` domain, it was considered acceptable to migrate the MediaWiki sites to GitHub Pages.

GitHub Pages uses [Jekyll](http://jekyllrb.com) to generate static web sites, and the most convenient input format for Jekyll is [markdown](http://daringfireball.net/projects/markdown/syntax), so the migration involved conversion from MediaWiki internal representation, to markdown. Given the differences, this conversion was not easy, and was done partly with scripts, partly manually. Similarly for MediaWiki, although the conversion from the MediaWiki format to markdown was easier. (Note: the migration is still work in progress).

The web site has two dedicated projects:

* [xpack.github.io](https://github.com/xpack/xpack.github.io) - the xPack web static site
* [xpack.github.io-source](https://github.com/xpack/xpack.github.io-source) - the complete Jekyll source for the xPack web

The first one stores the actual static pages of the Web site and the second stores the Jekyll source files for generating the Web site.

## Old sites

The previous XCDL site is still available from http://xcdl.github.io, but its content is no longer maintained.
