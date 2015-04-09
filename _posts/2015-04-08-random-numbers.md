---
layout: post
title: Liquid - Random Numbers
categories: [programming]
tags: [jekyll, liquid]
date: 2015-04-08 21:21:07-04:00

---

Refs:

* https://ecommerce.shopify.com/c/ecommerce-design/t/this-is-how-i-generate-random-numbers-with-liquid-201660
* http://computer.howstuffworks.com/question697.htm

Random numbers are hard to do well.  Well, hard to make non-repeating and evenly distributed.

Heres one way to do it in liquid:

```liquid
{ % capture time_seed % }{ { 'now' | date: "%s" } }{ % endcapture % }
{ % assign random = time_seed | times: 1103515245 | plus: 12345 | divided_by: 65536 | modulo: 32768 | modulo: 10 % }
```
