### git stash

有时候想要切换分支，但是还不想要提交之前的工作；可以使用 `git stash` 或 `git stash save`, 将新的储藏推送到栈上, 存储修改. 

要查看储藏的东西，可以使用 `git stash list`. 注意可能会有多次存储, 所以是list. 

`git stash apply`: 使用该命令, 可以将刚刚储藏的工作重新应用, 如果想要应用其中一个更旧的储藏，可以通过名字指定它，像这样：git stash apply stash@{2}。 如果不指定一个储藏，Git 认为指定的是最近的储藏.

可以运行 `git stash drop` 加上将要移除的储藏的名字来移除它

### 重写历史

很多时候，在 Git 上工作的时候，你也许会由于某种原因想要修订你的提交历史。你可以在你即将提交暂存区时决定什么文件归入哪一次提交，你可以使用 `stash` 命令来决定你暂时搁置的工作，你**可以重写已经发生的提交以使它们看起来是另外一种样子**。这个包括`改变提交的次序`、`改变说明`或者`修改提交中包含的文件`，`将提交归并、拆分或者完全删除`——这一切在你尚未开始将你的工作和别人共享前都是可以的。

* 改变最近一次提交

```
git add / rm your file
git commit --amend
``` 

这会把你带入文本编辑器，里面包含了你最近一次提交说明，供你修改。当你保存并退出编辑器，这个编辑器会写入一个新的提交，里面包含了那个说明，并且让它成为你的新的最近一次提交。

* 修改多个提交说明

Git没有一个修改历史的工具，但是你可以使用rebase工具来衍合一系列的提交到它们原来所在的HEAD上而不是移到新的上。依靠这个交互式的rebase工具，你就可以停留在每一次提交后，如果你想修改或改变说明、增加文件或任何其他事情。你可以通过给git rebase增加-i选项来以交互方式地运行rebase。你必须通过告诉命令衍合到哪次提交，来指明你需要重写的提交的回溯深度。


* 其他关于`重排提交`, `拆分提交`查看文档[工具-重写历史](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E5%86%99%E5%8E%86%E5%8F%B2)

### `reset`

`reset`操纵`HEAD` `Index` `Working Directory`这三棵树.

`reset` 做的第一件事是移动 HEAD 的指向。 这与改变 HEAD 自身不同（`checkout` 所做的）

如果想`修改上次提交内容`, 可以`reset`回`HEAD~`（HEAD 的父结点), 其实就是把该分支移动回原来的位置，而**不会改变索引和工作目录**。 现在你可以更新索引并再次运行 git commit 来完成 git commit --amend 所要做的事情了.

注意`--soft`(会重置HEAD, 保留和Index暂存区和工作区最后一次提交内容), `--mixed`(默认)(会重置HEAD和Index暂存区内容, 只保留工作区), `--hard`(危险, 会覆盖工作区内容, 相当于撤销了最后一次提交)的使用, 更多参考文档[Git重置](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)


正因如此, 如果提交内容到错误的分支上了, 可以用`reset`很方便的修正.
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



### 参考
[git文档](https://git-scm.com/book/zh/v2/)
