---
layout: post
title: Liquid - Sorting arrays
categories: [programming]
tags: [jekyll, liquid]
date: 2015/03/25 22:03
---
Ref:

* https://github.com/Shopify/liquid/pull/304
* http://jekyllrb.com/docs/templates/

Sorting arrays in liquid (used in jekyll theming processing)

parkr showed how to do this without the pull being accepted which is great!

{% capture text %}|.% assign sortedPosts = site.posts | sort_by:name %.|{% endcapture %}    
{% include JB/liquid_raw %}

sorting collections in liquid can be done as:
{% capture text %}|. |. page.tags | sort .| .| {% endcapture %}    
{% include JB/liquid_raw %}
