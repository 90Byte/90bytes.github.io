---
title: "Post: To check when a time-taking command is finished in the terminal"
last_modified_at: 2022-10-20T05:05:08+0900
categories:
  - Blog
tags:
  - Terminal
  - tput
---

## TL;DR

```<some cli command> && tput bel```

## How does it work?

`tput` is a terminal control service. if you run `tput bel`, it means that you ordered the terminal to ring the system
alarm sound.

## Where can I use it?

I use it when I have to download or upload large files using `scp` or something like that, and I want to know when it is
finished.

It's a relatively primitive solution yet, but pretty simple.