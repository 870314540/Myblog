##git 与 github

###简介





###命令及示例

- git status :查看代码的修改状态

```
git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   .DS_Store
	modified:   "\350\204\232\346\234\254/.DS_Store"
	modified:   "\350\204\232\346\234\254/curl.md"

no changes added to commit (use "git add" and/or "git commit -a")
```
红色或绿色部分字体是工程内的发生修改的状态标识:

`modified` 代表文件和上一版本相比，有过修改

`new  file`  代表文件是新增加的

`deleted`   代表文件被删除了，提交成功后，文件将从repository中删除

`untracked file` 一般都是新增加的文件夹


- git  init 

 git init 是初始化一个git仓库，比如新建一个demo1文件夹将它git初始化
 
 >git init –bare是初始化一个裸仓库，没有工作区的，只会记录git提交的历史信息

 [仓库与项目源码分离](https://blog.csdn.net/sinat_34349564/article/details/52442526)
 
 
- git diff \<filename> : 查看修改变化 
 
 查看两个版本的变化： 
 git diff \<hascode> \<hashcode> \<filename>  

 
- git add :暂存提交的文件

`git add --all ` 提交所有文件 

 
- git commit :提交已暂存的文件 



- git push 

`git push -u origin master `:push 到远程仓库 master分支

```
Counting objects: 7, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (7/7), 2.93 KiB | 2.93 MiB/s, done.
Total 7 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To github.com:…………………….git
   e573f67..b106583  master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```









###问题整理




