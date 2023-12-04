---
layout: post
title:  why df disk size bigger than du
categories:
- SRE
tags:
- 问题排查
- disk
---

**why df disk size bigger than du**
---

> keywords
>- block size
>- application stop
>
> df uses total allocated blocks, while du only looks at files themselves, excluding metadata such as inodes, which still require blocks on the disk. Additionally, if a file is deleted while an application has it opened, du will report it as free space but df does not until the application exits.
>
>[see stackexchange](https://unix.stackexchange.com/questions/9612/why-is-there-a-discrepancy-in-disk-usage-reported-by-df-and-du)


> how to find file deleted but space not freed
>
> `lsof +L1|grep deleted`
>


