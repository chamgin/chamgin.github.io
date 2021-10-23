---
title: "git rebase 命令"
date: 2021-10-20T18:00:50+08:00
---

&nbsp;&nbsp;&nbsp;&nbsp;git rebase 可以将当前的commit 转移到其他commit 的基础上，主要是改写了commit hisotry，因此可以解决很多问题，例如合并多个commit，将当前分支基于其他分支进行合并等。

----

### 使用rebase合并多个commit
&nbsp;&nbsp;&nbsp;&nbsp;当做一些项目调试时可能因为加临时代码会有多个commit, git log显得冗长，可以使用git rebase 在本地将多个commit 进行合并后再push上去。 
首先git log确定开始和结束的commit id范围，例如合并前3次commit可以这样操作：  
`git rebase -i HEAD~3`  
这样就会进入交互界面，在前两次使用squash可以合并到后一次commit，常用的操作有：  
| 命令 | 用途 |
| :---- | :---- |
| pick | 使用该commit |
| reword | 修改commit 消息 |
| squash | 保留该commit，但合并到上一个commit |
| fixup  | 类似squash，但去掉commit message |
| drop   | 删除该commit |

之后将后面的操作由pick变更为squash，然后:x(使用默认编辑器是vi的话)保存退出  
然后进入修改commit message的页面，可以考虑删掉前面几次的commit message，然后在最近的进行修改即可。完成后保存退出即可。  
之后使用git log即可看到自己的commit history被改写了，然后push即可。  

----


### 使用rebase在多个开发时解决冲突
&nbsp;&nbsp;&nbsp;&nbsp;日常开发过程中一般都是一个项目同个需求有多人合作开发，当开发完成代码后可能因为其他同事上传了已完成的代码导致出现某些冲突，或者想要将自己的代码基于同事的代码继续开发，都可以使用rebase进行合并。  
&nbsp;&nbsp;&nbsp;&nbsp;基于merge 的问题是会自动生成一个新的commit，并且在该commit会有两个parent，导致无论是比较代码还是查看历史不是很方便  
&nbsp;&nbsp;&nbsp;&nbsp;假设目前的开发节点如下：
```
                 B
                /
            ---X---A
	    希望变成
                     B'
                    /
            ---X---A
```
&nbsp;&nbsp;&nbsp;&nbsp;其中X横线是master分支，因此可以直接使用rebase功能，通过git rebase origin/master解决冲突后(如果有)，那么本地就会生成一个B'的新commit。  
&nbsp;&nbsp;&nbsp;&nbsp;并且这个commit是在其他人的需求代码之后的，因此也可以复用他人的功能。  
#### rebase后push 失败
&nbsp;&nbsp;&nbsp;&nbsp;由于rebase是更新了本地的commit history，但远程分支上的分支历史还是旧的，而git默认是使用fast-forward，也就是不允许push的commit history结构与远程不符合，导致直接git push失败。  
&nbsp;&nbsp;&nbsp;&nbsp;但是如果直接使用--force，则会可能因为其他人也刚好在push而导致他人push代码不生效，因此git提供了--force-with-lease 参数，该参数会检查提供的commit history与提交时是否保持一致进行保护，因此可以使用 git push --force-with-lease 进行提交   
