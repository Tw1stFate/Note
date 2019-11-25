### `reset`和`revert`的作用和区别

    git中，每一次提交都会生成一个commit.

    * git revert会生成一个新的commit，将之前的某个commit的修改恢复过来
    * git reset会将HEAD移动到某个commit上，换种说法就是将某个commit变成最后一个commit

### 我犯了严重的错误，怎么回到过去？

    ```
    git reflog
    ```

    你会看到你过去在所有分支上做的所有事情！
    每一个动作都有一个目录编号：HEAD＠{index}
    找到你犯错之前的那条记录
    `git reset HEAD@{index}`

    你可以用这个恢复误删的文件，或者，你做了一些修改，结果事情搞得一团糟，你可以秒回之前的状态。如果你做了一次不爽的合并(merge)，也可以这样反悔，总之你能瞬间回到事情都还OK的时候.

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
[Oh Shit, Git!?!](https://ohshitgit.com/)