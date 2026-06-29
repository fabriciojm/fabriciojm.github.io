---
title: "Listing my Latest Blog Posts on my GitHub Profile"
date: 2026-06-29T23:21:00+02:00
draft: false

tags: ["workflows", "writing"]
author: "Fabricio Jiménez Morales"

description: ""
showToc: false
TocOpen: false

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true

cover:
  image: ""
  alt: ""
  caption: ""
  relative: false
  hidden: true
---

In the last few months, I've been working extensively on my transition to Platform Engineering, and that includes my online profile. The very fact that you're reading this means, I may modestly say, that it's working!

Today, a short write-up about a nice improvement to [my GitHub Profile](https://github.com/fabriciojm): automatically listing [my latest blog posts](https://github.com/fabriciojm#writing). Since I use the Hugo Papermod theme, I had to make sure the site was producing an RSS feed. This is achieved by ensuring that `hugo.yaml` has `RSS` as an element of `outputs.home`. Additionally, I added the RSS icon to my header by adding the following element to `params.socialIcons`:

```
- name: rss
  url: "https://fabriciojm.github.io/index.xml"
```

Now that the site is producing an RSS feed, we can simply follow the instructions by Gautam Krishna R to add a workflow to the GitHub profile page: [https://github.com/gautamkrishnar/blog-post-workflow](https://github.com/gautamkrishnar/blog-post-workflow). They are easy to follow, and that's about it: a list of the last five posts appears on the profile page!

---

To be fair, this is nothing new, and it's fairly straightforward to figure out. I still find two valuable things about it:

- I wanted to copy something I like. There's something about reading other people's writing that you can't figure out in any other way. Hopefully, this will give you some insight into [how I learn and build](https://fabriciojm.github.io/posts/20260624-commonplace-repo/).
- I left the GitHub Actions workflow untouched. It has a parameter to set how many blog posts it will show (`max_post_count`), whose default value is five. When the result appeared on my page, the list couldn't show all my posts because (_drum roll_) I had already published six posts, and this would be the seventh. I got a small (but great) sense of accomplishment out of that.
