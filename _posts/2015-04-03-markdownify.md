---
layout: post
title: How to markdownify a liquid include
categories: [programming]
tags: [jekyll, liquid]
date: 2015-04-03 21:50:27-04:00

---

Refs:

* https://github.com/jekyll/jekyll/issues/1303
* http://stackoverflow.com/questions/27771508/showing-markdown-content-in-a-div

To include a markdown file in Jekyll rendering, you would use something similar to:

{% capture text %}|.% capture my_include %.||.% include a_markdown_file.md %.||.% endcapture %.|
|.|. my_include | markdownify .| .|
{% endcapture %}
{% include JB/liquid_raw %}

### Troubleshooting

During jekyll serving:

* Error: wrong argument type nil (expected String)
	- resolve by ensuring the "{ % capture my\_include" is **not** hyphenated.  Statements like "{ % capture my-include" will fail like this.  hyphenated capture var names not supported.
* Front-matter of the included markdown file is **not** required.  If you include it, the front matter will be rendered as markdown. (not interpretted json)
