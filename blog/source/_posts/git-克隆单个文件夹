---
title: 克隆仓库下单个文件夹
date: 2017-02-08 12:14:42
tags: git
---


git init test && cd test     //新建仓库并进入文件夹
git config core.sparsecheckout true //设置允许克隆子目录

echo 'tt*' >> .git/info/sparse-checkout //设置要克隆的仓库的子目录路径   //空格别漏

git remote add origin git@github.com:mygithub/test.git  //这里换成你要克隆的项目和库

git pull origin master    //下载
————————————————
版权声明：本文为CSDN博主「周大侠啊」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_36560161/article/details/78260532