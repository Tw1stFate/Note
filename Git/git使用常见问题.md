### git stash

有时候想要切换分支，但是还不想要提交之前的工作；可以使用 `git stash` 或 `git stash save`, 将新的储藏推送到栈上, 存储修改. 

要查看储藏的东西，可以使用 `git stash list`. 注意可能会有多次存储, 所以是list. 

`git stash apply`: 使用该命令, 可以将刚刚储藏的工作重新应用, 如果想要应用其中一个更旧的储藏，可以通过名字指定它，像这样：git stash apply stash@{2}。 如果不指定一个储藏，Git 认为指定的是最近的储藏.

可以运行 `git stash drop` 加上将要移除的储藏的名字来移除它

### `reset`

`reset`操纵`HEAD` `Index` `Working Directory`这三棵树.

`reset` 做的第一件事是移动 HEAD 的指向。 这与改变 HEAD 自身不同（`checkout` 所做的）

如果想`修改上次提交内容`, 可以`reset`回`HEAD~`（HEAD 的父结点), 其实就是把该分支移动回原来的位置，而**不会改变索引和工作目录**。 现在你可以更新索引并再次运行 git commit 来完成 git commit --amend 所要做的事情了.

注意`--soft`(会重置HEAD, 保留和Index暂存区和工作区最后一次提交内容), `--mixed`(默认)(会重置HEAD和Index暂存区内容, 只保留工作区), `--hard`(危险, 会覆盖工作区内容, 相当于撤销了最后一次提交)的使用, 更多参考文档[Git重置](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)

##### `reset`和`checkout`:

和 reset 一样，checkout 也操纵三棵树，不过它有一点不同，这取决于你是否传给该命令一个文件路径.

* 不带路径
运行 `git checkout [branch]` 与运行 `git reset --hard [branch]` 非常相似，它会更新所有三棵树使其看起来像 [branch]，不过有两点重要的区别。

    * 首先不同于 `reset --hard`，`checkout` 对工作目录是安全的，它会通过检查来确保不会将已更改的文件弄丢。 其实它还更聪明一些。它会在工作目录中先试着简单合并一下，这样所有_还未修改过的_文件都会被更新。 而 reset --hard 则会不做检查就全面地替换所有东西。

    * 第二个重要的区别是如何更新 `HEAD`。 `reset` 会移动 `HEAD` 分支的指向，而 `checkout` 只会移动 `HEAD` 自身来指向另一个分支。

    ![reset和co](./res/reset和co.png)

* 带路径
运行 `checkout` 的另一种方式就是指定一个文件路径，这会像 reset 一样不会移动 `HEAD`。 它就像 `git reset [branch] file` 那样用该次提交中的那个文件来更新索引，但是它也会覆盖工作目录中对应的文件。 它就像是 `git reset --hard [branch] file`（如果 `reset` 允许你这样运行的话）- 这样对工作目录并不安全，它也不会移动 HEAD。

此外，同 `git reset` 和 `git add` 一样，`checkout` 也接受一个 `--patch` 选项，允许你根据选择一块一块地恢复文件内容。

### 我犯了严重的错误，怎么回到过去？

```
git reflog
```

你会看到你过去在所有分支上做的所有事情！
每一个动作都有一个目录编号：HEAD＠{index}
找到你犯错之前的那条记录
`git reset HEAD@{index}`

你可以用这个恢复误删的文件，或者，你做了一些修改，结果事情搞得一团糟，你可以秒回之前的状态。如果你做了一次不爽的合并(merge)，也可以这样反悔，总之你能瞬间回到事情都还OK的时候.

注意: `git reset HEAD@{index}`并不会修改暂存区的内容.

### 我提交了，但立马想要做一个很小的修改！

```
# 没问题，你直接先修改好，然后:
git add .    # 或添加特定文件
git commit --amend
# 在出现的窗口里，编辑提交说明，或者维持不变
# 现在你最新的提交里已经包含了这个小修改
```

这种事经常发生，当我提交后，再进行测试或格式检查时，我在一个等号后面少了一个空格...其实你也可以把小修改当作一次新的提交，然后用rebase -i把它们合到一块儿，但上面的方法要快一百万倍.

### 提交说明写错了，怎么修改?

```
git commit --amend
# 在出现的窗口里编辑提交说明
```

### 我把应该提交给新分支的东西，提交给了主分支.

```
# 从当前状态的主分支上新建一个分支
git branch the-new-branch-name
# 取消主分支上最新的一次提交
git reset HEAD~ --hard
git checkout the-new-branch-name
# 好了，你的提交现在就只在这个新分支上
```

注意：如果你已经推送到了远程分支，那一切都晚了。无尽的忧桑...另外，如果你已经胡乱做了其他事情，那第二行应该是git reset HEAD@{number}，而不是HEAD~，以回到最初犯错的时间和地点.

### 提交到了错误的分支上

```
# 取消最新的提交，然后保留现场原状
git reset HEAD~ --soft
git stash
# 切换到正确的分支
git checkout name-of-the-correct-branch
git stash pop
git add .    # 或添加特定文件
git commit -m "你的提交说明"
# 现在你已经提交到正确的分支上了
```

也可以用`cherry-pick`, 但是上面的更简单.

### 我想用diff检查差别，但什么也没有

```
git diff --staged   //用这个
```


### 参考
[git文档](https://git-scm.com/book/zh/v2/)
[Oh Shit, Git!?!](https://ohshitgit.com/)