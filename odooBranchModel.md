## Odoo分支模型介绍  

[TEST](#test)
在阅读本文前需要了解[这些知识](./pre-git.md)

该分支模型将分支分为生产分支, 开发分支, 个人分支, 发布分支和热修分支  .
在代码库长久存在的只有生产分支和开发分支.   
生产分支旨在提供一个运行稳定, 功能完善的版本, 其上的提交记录不与开发分支关联, 主要是提供版本记录.  
开发分支上记录个人分支, 热修分支, 发布分支上的所有提交记录, 这些提交记录都要求使用快进式合并,
保证一个线性的, 直观简洁的提交记录.  
并且在对每一个分支的更新都要求的详细的测试流程, 以保证正常的工作流程.  
1. 个人分支的每一次整理后的提交, 都要进行单元测试, 保证针对这个提交相关的功能能正常运行, 使得开发分支回滚到任意一个提交都能正常运行.
2. 在个人分支变基到开发分支的最新节点后, 要求集成测试, 保证各模块之间能正常的协作运行, 不会导致其他模块产生错误.
3. 在完成热修分支和发布分支上的错误修复后, 要求进行回归测试, 保证没有因为修复错误而产生的新错误.
4. 发布分支在签出后, 要求进行一次完备的系统测试, 确保新版本能持续可靠的运行.


#### 生产分支/master分支  
生产分支旨在提供一个运行稳定, 功能完善的版本.  
其上的提交记录不与开发分支关联, 主要是提供版本记录.  
每一次生产分支的更新都要求经过系统测试, 保证没有没有影响用户使用的恶性错误.  

#### 开发分支/develop分支  
开发分支拥有最新的集成代码, 记录着每一次提交记录.  
为了报证这些提交记录的直观与简洁, 提出以下要求:
1. 每一次合并都是用快进式合并, 不留下分支的分叉信息.
2. 每一次提交都应该使用规范的提交格式.
3. 每一次提交都是有意义的.
4. 每一次提交包含且只包含一个独立的功能点.
5. 每一次提交都要经过单元测试, 保证每一次提交都可直接运行.

#### 个人分支/feature分支  
个人分支为开发者的日常开发分支, 由开发分支签出, 最后也合并到开发分支.  
个人分支的代码在合并之前需要提前解决与开发分支最新提交的合并冲突, 并通过代码审核与集成测试, 确保没有影响其他模块的正常运行.


1. 创建个人分支  

为了尽可能的避免冲突的产生, 在新建个人分支时应该从开发分支最新的提交上签出.   命名时需符合个人分支命名规范 __feature-$NAME__  
例如: feature-ben  

    ```
    # 更新本地开发分支, 如之前提到的有两种更新方式
    # 更新方式一
    git checkout develop
    git pull origin develop
    # 更新方式二
    git fetch origin develop:develop        # 注:该命令无法在 develop分支上执行.
    
    # 从最新的develop签出个人分支
    git checkout develop
    git checkout -b feature-ben
    ```

2. 进行开发  

在个人分支上完成手头的工作, 每一次提交的记录要求符合 [Git Commit规范](https://note.youdao.com/)  
为了数据安全和便于审核代码, 开发过程每天请至少进行一次提交  

    ```
    ############################################
    # 开发功能或者debug, 请保证每个提交包含且只包含一个独立的功能点.
    ############################################
    git add .
    git commit -m "[IMP] contact_extend: Some messages."

    git add .
    git commit -m "[IMP] contact_extend: Some messages."

    git add .
    git commit -m "Save code."

    git add .
    git commit -m "[IMP] contact_extend: Some messages."

    git add .
    git commit -m "[IMP] contact_extend: Some messages."
    
    # 查看提交记录
    git log
    ------- 输出 -------
    commit b4rg4d6...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.
    commit 4b464d6...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.
    
    
    commit 4b123d6...347 
    Author:
    Date:
    Save code.

    
    commit 9daw223...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.
    commit 09sef33...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.
    -------------------
    
    ############################################
    # 对于无意义的提交记录, 请将其删除或合并到与其相关的提交中.
    # 例如上面的的第二次提交 'Save code'.
    ############################################
    git rebase -i develop
    
    ------- 输出 --------
    pick 4b464d6 [IMP] contact_extend: Some messages.
    
    # pick 4b6546d Some code. 将 pick 修改为 fixup.
    fixup 4b6546d Some code.
    
    pick 9daw223 [IMP] contact_extend: Some messages.
    pick 09sef33 [IMP] contact_extend: Some messages.
    ---------------------
    # 保存退出
    
    
    git log
    
    ------- 输出 -------
    commit 12364d6...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.
    commit 4b464d6...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.
    
    
    # 此处的'Save code`那次提交就被合并到父提交了.
    
    
    commit 9daw223...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.
    commit 09sef33...347 
    Author:
    Date:
    [IMP] contact_extend: Some messages.

    --------------------
    
    ```

![创建个人分支](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/createPersonalBranch.png)  

[如图3.1](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/createPersonalBranch.png)
中我从 `develop` 分支的最新节点[dev-p1] 签出了我们的个人分支 `feature-ben` , 并已经在个人分支上提交了四次经过整理的`commit`记录.  
此时在`feature-ben`分支上的`commit`记录应该为图3.1中右侧所示包含五个`commit`记录, 并且基点为`dev-p1`.  
假设这四次`commit`记录, 已经完成了我们当前部分的开发, 即需要将该部分代码合并到`develop`分支. 

3. 提交合并请求  

<span id="test">test</span>
以上我们已经创建了我们自己的开发分支, 并且完成了一次开发, 现在我们需要将我们的新功能合并到开发分支上.  
但在那之前, 我们还需对我们这次开发的内容变基到远程 develop 分支最新提交上以解决可能在合并时产生的冲突,  
最后还需要进行代码审核以及单元测试.  

    ```
    git fetch origin develop:develop
    git rebase develop                  # 如果你了解了 git pull --rebase 的用法, 也可用它代替.
    
    # 更新远程个人分支
    git push origin feature-ben
    
    ```
![rebase个人分支](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/rebasePersonalBranch.png)  

 

首先需要先更新`develop`分支, 假设更新完成后develop分支的提交记录
[如图3.2](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/rebasePersonalBranch.png)所示.   
我们发现已经有人提交了多个记录在`develop`分支上, 在图中表现为四次紫色的针对`sale_quotation`模块的`[IMP]`类型的提交.  
为了将`feature-ben`分支上的新功能合并到`develop`分支上, 我们需要将`feature-ben`分支变基到`develop`分支上的最新提交节点`sale_quotation_new_fucntion`.  
即将`feature-ben`的基点由`dev-p1`修改到`sale_quotation_new_fucntion`,   
而此时`develop`正指向`develop`分支上的最新提交`34324e::sale_quotation_new_function`, `34324e`为`sale_quotation_new_function`这次提交的`commit_id`.  
因此在变基时我们可以使用`git rebase develop`或者`git rebase 34324e`.  

在以上rebase的过程中, git会在最新节点`34324e`后应用你的每一次提交, 此处我的功能只有四个提交, 因此会应用四次提交.  
当我的某一次提交(红)与紫色的提交内容产生冲突时, git 就会停止rebase, 并抛出一个文件冲突的异常.
到此, 我们需要手动修复这些冲突, 修复完成后使用 'git add .' 来跟踪修复产生的变动.
大部分情况下,  git会使用类似于以下代码块的方式来标注冲突位置.
```

>>>>>>> HEAD
被他人修改的代码.
=======
你修改的代码.
<<<<<<< [IMP] contact_extend: ....

```
但也存在部分特殊情况:
a. 直接删除或重命名了代码文件, 该行为git会抛出冲突异常, 但无法找到冲突位置.  
解决此类冲突时较为复杂, 首先确定哪些文件被删除或重命名, 然后确定自己的修改是否涉及到这部分代码,  
    - 如果没有, 则手动对该部分文件进行删除或重命名即可解决这次冲突;  
    - 如果修改了该部分代码, 怎应该了解该部分文件为何被删除或重命名, 基于此修改你的代码.  

面对这些冲突, 我们需要找出多人修改同一个文件的原因, 并确定具体应用那一次提交的变动. __禁止修改或删除他人的代码!__   
修复完冲突后, 需要使用`git add`来跟踪修复产生的变动.  

只有在父提交应用成功无冲突或解决了冲突后才能继续使用`git rebase --continue`应用下一次提交.  
变基后`feature-ben`分支的`commit`记录应该变为如图3.3所示.   
移动基点后解决了 feature-ben 与 develop 的分歧, 现在我们可以使用快进式合并合并两个分支.  

此时我就完成了我的代码合并工作, 可以向远程分支`origin/feature-ben`分支提交我的代码.   
但是在提交时, 由于我们的提交记录中多出了一些本不存在的`commit`记录, 即图中紫色的提交,  
因此无法推送. 但我们可以使用`--force 或 -f`标识来进行强制推送.  

推送到远程仓库以后, 还需要进行代码审核以及将代码合并到`develop`等操作.  
当`develop`分支合并了我们代码后, 它的`commit`记录应该如图3.4所示.  



#### 发布分支/release 分支  

当我们的开发进行到某个里程碑时, 我们可能需要这些新功能发布到生产分支上, 此时我们就需要创建一个发布分支.  
在该分支上进行发布前的系统测试确保各模块功能之间能正确的协调工作而没导致新的错误.  
在系统测试发现错误的情况下, 直接在该分支上进行修复.   
如果需要多人合作开发, 则在发布分支的基础上签出个人发布分支并命名为`release-$USER_NAME`. 之后确保将个人发布分支使用快进式合并到发布分支.  
完成修复后还需要在该分支上进行一次回归测试, 确保没有因修复而产生的新错误.  

最后将该分支合并到开发分支和主分支.

![createReleaseBranch](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/createRealeseBranch.png)  

在前文中我已经将feature-ben分支合并到develop分支, 假设这些提交满足了我们发布新版本的条件,  
现在我们就可以从develop 分支上签出一个release分支, 在图4.1用黄色表示.
```
git fetch origin develop:develop        # 更新develop 分支
git checkout 6e734k                     # 切换到可以发布新版本的那次提交上
git checkout -b release                 # 从当前提交签出一个发布分支

############################################
# 在发布分支上测试和debug
############################################
git add .
git commit -am '[REL] Some messages'    # 假设我们在测试的过程中发现了某些新的问题, 我们可以直接在发布分支上进行修复.
                                        # 如果发现的问题需要多人合作开发时, 请从发布分支上签出新的分支并命名为 release-$USER_name, 例如 release-ben.
                                        # 当发布分支上的问题修复完成时, 如图4.2同样使用 rebase 的方式来确保我能使用快进式合并的方式将 release-ben 合并到 release 分支上.
                                        # 解决完这些发现的问题后, 需要在release分支上进行回归测试, 防止我们修改的些bug引起新的问题.


# 合并到生产分支
git checkout master
git merge --squash release              # 如图4.1, 当我们解决了发布分支上的所有问题后, 我们使用 git merge --squash release 来将release分支上的所有变动应用到master分支上.
git commit -am '[REL] Some messages'    # 现在使用 git commit -am 来跟踪这些应用的变动.
git tag 1.1.0                           # 最后需要为这一次提交打上版本标签.

# 合并到开发分支
git checkout release-1.0.0              # 如图4.3, 合并到开发分支时同样使用 rebase 方式来确保使用快进式合并.
git pull origin develop --rebase        # 假设我们在修复发布分支上的问题时, 开发分支上又有了新的提交, 在图4.3中用紫色表示.
                                        # 那么为保证开发分支的提交记录直观性和发布分支具体的发布点的正确性. 
                                        # 我们将开发分支上的新提交(三个紫色的提交) rebase 到发布分支的最新提交上, 如图4.4.
                                        # 最后进行合并, 得到开发分支的提交记录如图4.5所示.
                                        
# 删除发布分支
git branch -d release                   # 因为我们已将发布分支上的提交记录, 全都同步到了开发分支上, 在图4.5中变现为黄色和橙色, 所以删除发布分支.

# 推送更新到远程仓库
git checkout develop
git push origin develop

git checkout master
git push --tags origin master
```



#### 热修分支/hotfix 分支  

当生产分支的在代码在使用过程中发现了某些十分严重, 影响系统运行的恶性bug时, 我们需对其进行紧急的修复.   
这时, 我们就需要创建热修分支. 当修复这些bug需要多人合作进行时, 应在热修分支的基础上签出新的个人热修分支, 并命名为 __hotfix-$USERNAME__.  
个人热修分支修复完成后, 也需要回归测试, 防止新的bug产生.  
最后发布分支一样使用`rebase`的方式来确保使用快进式合并来将热修分支合并到开发分支, 使用`squash`的方式合并到主分支.  

![createHotfixBranch](https://github.com/alreadytakenn/images/blob/master/gitBranchModel/createHotfixlBranch.png)  

```
git fetch origin master:master          # 更新主分支
git checkout master                     # 切换到主分支
git checkout -b hotfix                  # 签出热修分支

# 修复紧急bug
git add .
git commit -am '[FIX] crm_extend: Some messages.'
                                        # 此处与发布分支类似, 如果多人合作开发时就使用rebase确保快进式合并.

# 完成所有修复后, 需要在热修分支上进行回归测试, 防止因修复产生的新错误.

# 合并到生产分支
git checkout master
git merge --squash hotfix
git commit -am '[FIX] Some messages.'
git tag 1.1.1                           # 此处与发布分支类似, 应用所有变动到主分支, 在进行跟踪, 合并, 打标签.

# 合并到开发分支
git fetch origin develop:develop        # 更新开发分支, 在我们进行热修的同时, 可能开发分支已经有了更新的提交. 在图5.1中表现为蓝色提交.
git checkout hotfix                     
git rebase develop                      # 这里区别于发布分支, 因为热修分支是从主分支签出, 如图5.1 其基点是1.0.0, 我们将其变基到开发分支的最新节点, 如图5.3 
git checkout develop
git merge hotfix                        # 合并的结果如图5.4所示.

git branch -d hotfix                    # 删除热修分支, 因为热修分支上所有的提交已经应用到开发分支.

# 推送更新到远程仓库
git checkout develop
git push origin develop

git checkout master
git push --tags origin master
```



```
git fetch origin master:master          # 更新主分支
git checkout master                     # 切换到主分支
git checkout -b hotfix                  # 签出热修分支

# 修复紧急bug
git add .
git commit -am '[FIX] crm_extend: Some messages'

[可选] git rebase -i develop 
############################################
# 清理合并不相关的提交记录
############################################

# 合并到生产分支
git checkout master
git cherry-pick hotfix-1.0.1

# 合并到开发分支
git checkout develop
git cherrt-pick -x $start_commit_id ^.. $end_commit_id

```
