---
layout:     post
title:      CI加载library的问题
subtitle:   CI+MX构建项目
date:       2017-11-08
author:     sjhh
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - php
    - CI
---
最近遇到一个bug，仔细研究了下CI下load library的代码，为方便自己以后查阅，记录与此。
现象：代码里重复load同一个library的时候，会出现load失败的现象。
使用load->library('class', $params, $alias)函数加载。
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

