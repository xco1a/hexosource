---
title: hexo+github搭建过程中的命令
date: 2018-05-18 09:38:30
tags: 
- 瞎折腾
categories: 
- 博客
---

&emsp;&emsp;简单记录一下折腾博客学到的命令。git还需要花点时间系统的学习一下。        

### git 批量删除和提交              

​	git rm * -r      (-r 递归删除)                 

​	git commit -m "clear"                

​	git push (推送到远程仓库)               

### hexo本地部署（需要node.js环境）             

​	npm install hexo-cli -g              

​	hexo init blog          

​	cd blog            

​	npm install            

​	hexo clean            

​	hexo g          

​	hexo d       

#### hexo主题next                

​	git clone https://github.com/iissnan/hexo-theme-next themes/next              