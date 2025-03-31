---
layout: post
title: Test Markdown
subtitle: This post test self markdown. 
tags: [guide, markdown]
---

This post is written in markdown, but you can also write a [post using html]({% link _posts/2020-02-27-html-posts.html %}).

<span class="color-red">[NEW]:</span> Now you can also create a private post, which will not be visible on the blog homepage, but is accessible via a URL. See [this secret post]({% link _posts/2020-04-05-private-example.html %}) as an example.

Posts should be named as `%yyyy-%mm-%dd-your-post-title-here.md`, and placed in the `_posts/` directory. Drafts can be kept in `_drafts/` directory.

-------------

# 标题1
## 中文标题2
### This is a sub-sub-heading

<span class="color-blue">蓝色</span>
<span class="color-green">绿色</span>
<span class="color-orange">colorful</span>
<span class="color-red">text.</span><br>

<span class="highlight-blue">And</span>
<span class="highlight-green">some</span>
<span class="highlight-orange">橙色</span>
<span class="highlight-red">styles.</span>

**Here is a bulleted list,**
 - This is a bullet point
 - This is another bullet point


**Here is a numbered list,**
1. You can also number things down.
2. And so on.

**Here is a sample code snippet in C,**
```C
#include <stdio.h>

int main(void){
    printf("hello world!");
}
```

**Here is a horizontal rule,**

--------------

**Here is a blockquote,**

> 测试引用
> circumstances of your life can change!

**Here is a table,**

ID  | Name   | Subject
----|--------|--------
201 | John   | Physics
202 | Doe    | Chemistry
203 | Samson | Biology

**Here is a link,**<br>
[GitHub Inc.](https://github.com) is a web-based hosting service
for version control using Git

**Here is an image,**<br>
![](../assets/autumn.jpg)

