---
layout: post
title: Finding theme elements
categories: [programming]
tags: [jekyll, liquid]
---

Ref:

* http://jekyllrb.com/docs/variables

Trying to become an expert on any new theme in jekyll is a bit awkward.  Noticing that each theme tends to make up thier own customizations - some consistent with others... some not so much.

To find some of them front-matter worthy; try this:

```bash
egrep -ohR "site\.\w*\.?\w*?" | sort | uniq
```

gives something like this:

```text
site.atom_path
site.author.email
site.author.feedburner
site.author.github
site.author.name
site.categories
site.JB.analytics
site.JB.archive_path
site.JB.ASSET_PATH
site.JB.atom_path
site.JB.BASE_PATH
site.JB.categories_list
site.JB.categories_path
site.JB.comments
site.JB.liquid_raw
site.JB.pages_list
site.JB.pages_path
site.JB.posts_collate
site.JB.rss_path
site.JB.setup
site.JB.sharing
site.JB.tags_list
site.JB.tags_path
site.pages
site.posts
site.production_url
site.related_posts
site.rss_path
site.safe
site.static_files
site.tagline
site.tags
site.tags.cool_tag
site.time
site.title
site.var.tags_path
```