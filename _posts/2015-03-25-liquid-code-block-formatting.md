---
layout: post
title: Liquid code block formatting
categories: [programming]
tags: [jekyll, liquid]
---


Ref:

* http://truongtx.me/2013/01/09/display-liquid-code-in-jekyll/

Formatting liquid code under jekyll using pygment code block is a pain.  Mainly due to the \{ \} and %'s.  The Jekyll bootstrap has some liquid raw support for this built-in:

1. Get your liquid code wanted for posting...
2. Replace all the \{ with |.
3. Replace all the \} with .|
4. wrap the result in
	\{ % capture text % \} \[your code here \]\{ % endcapture % \} 
5. include the JB magic... \{ % include JB/liquid_raw % \}

Example:

{% capture text %}|.% assign sortedPosts = site.posts | sort_by:name %.|{% endcapture %}    
{% include JB/liquid_raw %}
