---
title: hexo-operation
date: 2021-11-05 16:54:14
tags: [hexo, tool]
categories: 
	- code-tool
	- hexo
---

1. 新建
`hexo new "file-name"`
新建的markdown文件在`source/_post/`目录下

2. 生成静态网页
`hexo generate`
生成的静态网页在`public`目录的相应的日期下，比如2021-11-05生成，在`public/2021/11/05`文件夹下

3. 本地启动
`hexo server`
启动端口为4000，可使用`localhost:4000`查看

4. 部署
`hexo deploy`
网址为：huiresurgeran.github.io
带提交信息`hexo deploy -m "commit messgae`

5. 删除
（1）去本地文件夹的`/source/_post`目录下删除需要删除的`.md`文件
（2）去本地文件夹的`/public`目录下删除这篇博客对应的文件夹（根据发布时间归档）
（3）重新生成并发布：generate + deploy

6. 问题
博客更新时出现问题，可以进行清理并重新生成静态网页
`hexo clean`
`hexo generate`

7. 参考内部文档
https://km.woa.com/group/19976/articles/show/297274?kmref=search&from_page=1&no=2
