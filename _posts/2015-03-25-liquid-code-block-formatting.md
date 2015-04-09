---
layout: post
title: Liquid code block formatting
categories: [programming]
tags: [jekyll, liquid]
date: 2014-07-14
updated: 2015-04-08 21:31:10-04:00
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

## Another Way
Yeah... the result can be difficult to accomplish.  I personally like ways that keep the text markdown very close to the rendered html.  You can accomplish the same affect by code fencing and putting a space between the \{.
Note: In code fences, we don't need to escape the curly.

Although - the spaces are annoying at least the are consitent; unlike the above JB magic.  and the spaces also throw off the pygment syntax highlighting.  its a fair compromise for cleaner text.