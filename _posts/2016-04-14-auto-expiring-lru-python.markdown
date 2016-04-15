---
layout: post
title:  "Auto-expiring LRU cache implementation in python -- draft"
date:   2016-04-14 23:50:00
categories: main
tags :
     - programming
     - python
---

Recently, I need to implement a rate-limiter for one of my server application. One of the important components used by rate-limiter is **auto expire key** cache. 

{% gist haocs/a355a2c90edc3d3508df91c9b10c6263 %}  
[Flask rate-limiter](http://flask.pocoo.org/snippets/70/)  
[Redis expire](http://redis.io/commands/expire)  
[my cache implementation on Github](https://github.com/haocs/algorithm_practice/blob/master/libs/cache.py)  
