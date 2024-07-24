+++
title = 'Git出现push报错'
date = 2023-09-14T20:53:53+08:00
draft = false
categories = ["Tech"]
tags = ["Git", "Tech"]
+++


当Git Bash 使用push的时候出现了以下的这个报错的，可以尝试下使用一下命令

`How to solve this problem of "! [rejected] master -> master (fetch first)"`

```
First Do this ...

git fetch origin master
git merge  master

Then, do this ...

git fetch origin master:tmp
git rebase tmp
git push origin HEAD:master
git branch -D tmp

Now everything works well.
```
<!--more-->

是在GitHub Gist中有人提出的

>  [GitHub Gist问题帖](https://gist.github.com/sharbel93/ebcf0b18782573f4d95f80caa3c84acb)




