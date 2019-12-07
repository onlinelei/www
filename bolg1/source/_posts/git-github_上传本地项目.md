---
title: github 上传本地项目
date: 2017-02-06 14:42:03
tags: Git
---


	//本地文件夹中新建本地仓库
	git init
	//新建文件
	vi README.md
	//将新建的文件添加到本地仓库中
	git add README.md  
	git commit -m "first commit"  
	//git@github.com 可以用别名替代
	git remote add origin git@github.com:onlinelei/test.git  
	git push -u origin master
