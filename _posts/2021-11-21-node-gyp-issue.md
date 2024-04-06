---
title: "Post: when node-gyp can't find source files"
last_modified_at: 2021-11-21T13:10:02+0900
categories:
  - Blog
tags:
  - Node.js
  - node-gyp
---
Lately, I've been developing a node addon as a hobby after work. I should be working hard on it, but it's so busy until the end of the year, this blog might freeze again...

Anyway, while developing the node addon and fiddling with binding.gyp like creating a makefile, suddenly, an error pops up saying there's nothing exported from the built module... 

I tried changing only the NODE_API_MODULE init part in the source code endlessly, but it was completely useless. 

I noticed something was wrong because even when I intentionally caused a compile error, the node-gyp rebuild still passed...


Eventually, I found out the reason was because I used a wildcard (\*) in sources thinking it would make the build a bit easier... It was a pointless effort, especially since there was only one source. Naturally, there were no built sources, hence no module export, and, naturally, since nothing was compiled, the build just passed.

Anyway, the reason why it didn't work is that gyp itself doesn't support wildcards for some reason...

Instead, since executing shell scripts is possible in gyp, it's indirectly possible through methods using ls like [this](https://stackoverflow.com/a/15419176).

The end!