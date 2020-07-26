> 介绍分支之前, 需要了解一下内容.  

1. 远程分支和本地分支
2. `fetch`和`pull`
3. 几种不同的合并方式
4. `rebase`和`rebase -i`
5. `rebase`和`merge`


### 远程分支与本地分支  

```
git branch -a

* hotfix                                    # '*'号标记的为当前所在的位置, 与头指针HEAD同意
  develop                                   # develop分支的本地克隆
  master                                    # master分支的本地克隆
  remotes/origin/HEAD -> origin/master      # 远程仓库中的头指针在本地仓库中的引用, 指向默认分支
  remotes/origin/hotfix                     # 远程hotfix分支在本地仓库中的引用
  remotes/origin/develop                    # 远程develop分支在本地仓库中的引用
  remotes/origin/master                     # 远程master分支在本地仓库中的引用
                                            # 这些远程分支无法移动, 当你将改动推送到远程仓库时, 引用会自动移动.
                                            # 别人更新了远程仓库内容时, 引用不会自动移动, 需要手动使用 git fetch origin <branch_name>
```

分支并不是指某一系列的提交或某一条线, 应该将其也理解为一个指向某次提交的标记或指针.  
在使用`git log`查看提交日志时, 这一点变得十分明显.  
在下面的提交记录中, 有三个本地分支(HEAD, hotfix, master), 两个个远程分支(origin/master, origin/HEAD)和一个标记(tag:1.0.1)指向了在`552e34`这次提交.  
其中`HEAD`指向当前位置, 所以指向了`hotfix`.  
```
git log

commit 552e34...347 (HEAD -> hotfix, tag:1.0.1, origin/master, origin/HEAD, master)
Author:
Date:
    [FIX] commit messages.

commit 545k23...if2 (tag:1.0.0)
Author:
Date:
    [FIX] commit messages.
    
...

```


### `fetch` 和 `pull`  
`git fetch origin develop`只会拉取远程仓库`origin`的`develop`分支中的到本地的`origin/develop`分支.  
`git pull origin develop`在拉取更新后还会将`origin/develop`分支合并到你当前所处的分支位置.  

```
git pull -r origin develop      # 将当前分支变基到远程develop分支的最新提交.
                                # -r == --rebase[=false|ture|merges|preserve|interactive]
                                # ture: 将当前分支变基到远程develop分支的最新提交.
                                # false: 直接合并
                                # merges: 在rebase中保留本地的merge commit, 没有分叉纪录.
                                # preserve: 在rebase中保留本地的merge commit, 保留本地分叉纪录. 
                                # interactive: 交互模式.
```


### 几种不同的合并方式  
- fast-forward  
```
git merge --ff develop      # 将develop分支以快进式合并的方式合并到当前位置(无分歧时的默认选项)
```
快进式合并, 当你的合并对象时当前位置的直接上游时, Git只是简单的向前移动指针.  
不产生新的commit记录和分叉记录. 这种方式能生成简洁直观的提交记录, 因此希望合并时都使用这种方式.    
但在合并对象与当前位置出现分歧时, 则无法使用该方式合并. 因此我们时`rebase`的方式来解决分歧.
- no-fast-forward (--no-ff)  
```
git merge --no-ff develop      # 不是使用快进式将develop分支合并到当前位置(有分歧时的默认选项)
```
不使用快进式合并, 该方式会保留你的分支信息与合并记录, 但使得提交记录变得复杂, 因此我们不希望使用该种方式合并.
- squash (--squash)  
```
git merge --squash develop      # 将develop分支以squash合并的方式合并到当前位置
```
Squash方式合并, 会将合并对象的所有变动都应用到当前分支, 然后需要手动进行`add`和`commit`操作.  
该方式会保留分支信息, 不会产生合并记录, 但会产生新的提交记录.


![不同的合并方式](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/waysofMerge.png)  
[不同的合并方式](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/waysofMerge.png)


### `rebase -i` 和 `rebase`  
*`rebase`之前一定要更新你的目标基点, 否则需要重新`rebase`的最新节点.*
```
git fetch origin master             # 更新origin/master
git rebase -i origin/master         # 将当前分支以交互模式变基到origin/master分支所指向的节点.
                                    # 这里一定要区别 git rebase -i master 和 git rebase -i origin/master
                                    # 因为 git fetch origin master 不会更新 master分支.
                                    # 如果你希望使用 git rebase -i master, 那么你需要先将远程master的变动更新到本地 master分支中.
                                    # 可以使用 git fetch origin master:master 或者 git checkout master && git pull origin master
                                    #
git rebase -i 4804dg                # 若origin/master指向:4804dg:这个commitID, 则可使用commmitID代替origin/master
```
简单的理解就是: `rebase -i` 为交互模式  
交互模式在`rebase`的过程中提供了交互界面用于整理我们当前分支的`commit`记录.  
使用`git rebase`即等同于`git rebase -i`后不在交互界面做任何改动直接保存退出.  
以下为`rebase -i`过程中可使用的命令.  
```
# p, pick <commit> = 应用当前提交
# r, reword <commit> = 应用当前提交, 但修改当前提交信息
# e, edit <commit> = 应用当前提交, 稍后进行修改提交消息 通过使用git rebase --edit-todo
# s, squash <commit> = 应用当前提交但将其合并到上一个提交中
# f, fixup <commit> = 类似于squash, 但会忽略当前提交信息, squash则会让你选择是否删除
# x, exec <command> = 将提交消息当作脚本执行
# b, break = 暂停rebase, 通过`git rebase --continue`继续  
# d, drop <commit> = 删除该提交
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
```
![分支的变基](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/rebasePersonalBranch.png)  
[分支的变基](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/rebasePersonalBranch.png)

`rebase`用于变基合并.  
即将当前分支的基点变到指定的commit记录.  
例如图(分支的变基)左侧是我们签出了一个新分支`feature-ben`, 并进行了四次提交.  
当前分支`feature-ben`的基点是`dev-p1`, 而当前`develop`分支的最新节点是`34324e`,   
因此当我们希望将当前分支`feature-ben`合并到`develop`分支时, 我们需要先更新本地的`develop`分支,  
然后在`feature-ben`分支上进行`git rebase develop`操作. 完成该操作后, `feature-ben`分支上的提交记录应变为如图(分支的变基)右侧所示.


### `rebase`和`merge`  
`rebase`变基与`merge`合并, 两者没有什么关系, 只是在合并的时候我们希望统一使用快进式合并, 而为了解决分歧时无法进行快进式合并这个问题, 我们使用了`rebase`来解决这个问题.  

合并`merge`有三种方式`--fast-forward <--ff>, --no-fast-forward <--no-ff>, --squash`, 其中`--no-ff, --squash`方式合并会保留
我们的分支上的提交消息, 因此会留下分叉的提交消息记录. 而当我们合并时, 若产生图(分支的变基)左侧所示情况, 无法避免的会使用`--no-ff`的方式合并.  
为了不保留这些分叉记录, 我们使用`rebase`的方式将`feature-ben`分支上的提交记录修改为图(分支的变基)右侧所示, 这种情况下提交, 即可使用`fast-forward`的方式合并.