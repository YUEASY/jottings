## git基本操作

### 1. 基础认识

##### 三个分区

1. 工作区：就是我们平时编写代码的目录。
2. 暂存区：也叫索引区，是一个临时保存我们修改内容的区域，⼀般存放在 `.git` 目录下的 `index` 文件`.git/index`中。
3. 版本库：也叫仓库，它是 Git 用来保存项目历史版本的地方，每次提交到版本库中的文件都会在`.git`生成一个唯一标识，以便于版本控制和管理。

##### Git 的基本工作流程

在工作区中进行文件的修改和编辑，
然后将需要提交的文件使用 `git add` 命令添加到暂存区，
最后使用 `git commit` 将暂存区的文件提交到版本库中，
若需要上传到远端仓库则使用 `git push` 命令。

### 2. 本地操作

#### part 1

##### 初始化本地仓库

```shell
git init 
```

##### 配置用户名和e-mail

```shell
git config [--global] user.name "Your Name" 
git config [--global] user.email "email@example.com"
# [--global]: 可设置全局更改
```

##### 查看配置

```shell
git config -l
```

####  part 2

##### 添加（工作区->暂存区)

```shell
# 添加一个或多个文件到暂存区： 
git add [file1] [file2] ...
# 添加指定目录到暂存区，包括子目录
git add [dir]
# 添加当前目录下的所有文件改动到暂存区
git add .
```

##### 提交（暂存区->版本库）

```shell
# 将暂存区全部内容提交到版本库中
git commit -m "message"
# 将暂存区的指定文件提交到版本库
git commit [file1] [file2] ... -m "message"
# message: 对本次提交的描述，不允许省略
```

##### 查看提交记录

```shell
git log [--pretty=oneline]
# --pretty=oneline :以漂亮的方式打印
```

##### 查看仓库的状态

```shell
git status
# 未修改/未添加/未提交
```

##### 查看暂存区和工作区的差异

```shell
git diff [file]
```

##### 查看本地仓库和工作区的差异

```shell
git diff HEAD -- [file]
# 注意格式！
```

#### part 3

##### 版本回退

```shell
git reset [--soft | --mixed | --hard] [HEAD]
# 可选项，默认为 --mixed
# 1. --soft:仅回退版本库到某一历史版本
# 2. --mixed:回退版本库和暂存区
# 3. --hard:全部回滚

# HEAD 说明:
# 1.可直接写成commit id，表⽰指定退回的版本
# 2.HEAD表示当前版本
# 3.HEAD^或HEAD~1表示上⼀个版本
# 4.HEAD^^或HEAD~2表示上上个版本
# ......
```

##### 查看提交的历史记录

```shell
git log [--pretty=oneline]
# 含每次提交的commit id
# --pretty=oneline :以漂亮的方式打印
```

##### 根据commit id查看提交信息

```shell
git cat-file [-p] [commit id]
```

##### 查看命令的历史记录

```shell
git reflog
# 含每次命令操作的短commit id，可作为commit id使用
```

##### 丢弃工作区的修改（恢复文件）

```shell
git checkout -- [file]
# 注意格式！
```

##### 撤销暂存区的修改

```shell
git reset HEAD
```

##### 撤销版本库的修改

```shell
git reset --hard HEAD^
# 未git push到远端的情况下
```

##### 删除工作区与暂存区的文件

```shell
rm [file]
git add [file]
# 或
git rm [file]
```

### 3.  分支管理

#### part 1

##### 查看所有分支

```shell
git branch [-a]
# 带*号的为当前分支
```

##### 创建新分支

```shell
git branch 分支名
```

##### 切换分支

```shell
git checkout 分支名
```

##### 合并分支1

```shell
git merge 需要合并的分支名
# 需要切换到目标分支
# 如:将dev合并到master
git checkout master
git merge dev
```

##### 合并分支2

```shell
git merge --no-ff -m "描述" 分支名
# 以非Fast-forward方式提交
# 当一个分支的提交历史是线性的，且当前分支没有新的提交时，Git 会使用 Fast-forward 合并策略。
# 该策略会快速将当前分支指向另一个分支的最后一个提交，完成合并操作，这个过程也被称为“快进合并”。
```

##### 删除分支

```shell
git branch -d 分支名
# 需要切换到其他分支
```

##### 强制删除分支

```shell
git branch -D 分支名
# 已经commit到仓库但未合并的分支会被git保护
```

#### part 2

##### 解决冲突

```shell
git merge 分支名
# 发生冲突
# 需要手动修改冲突的文件
# 重新添加和提交
git add .
git commit -m "描述"
# 冲突解决
```

##### 查看分支日志

```shell
git log --graph --abbrev-commit
# --abbrev-commit :打印缩写
```

#### part 3

##### 暂存工作区修改

```shell
git stash
# 将工作区的修改存入 ./.git/refs/stash
# 无法保存新增文件
```

##### 查看暂存的修改

```shell
git stash list
```

##### 恢复修改

```shell
git stash pop
```

### 4. 远程管理

#### part 1

##### 生成ssh密钥

```shell
ssh-keygen -t rsa -C "你的账户(邮箱)"
# user/.ssh下生成的id_rsa.pub文件就是公钥
```

##### 克隆远程仓库

```shell
#http
git clone https://github.com/....
#ssh
git clone git@github.com:.....
```

##### 查看相对应的远程库信息

```shell
git remote [-v]
# -v查看权限
```

##### 推送：将本地的分支版本上传到远程并合并

```shell
git push <远程主机名> <本地分支名>:<远程分支名>
# 如果本地分支名与远程分支名相同，则可以省略:
git push <远程主机名> <本地分支名>
# 远程主机名通常为origin
```

##### 拉取：从远程获取代码并与本地的当前分支并合

```shell
git pull <远程主机名> <远程分支名>:<本地分支名>
# 如果本地分支名与远程分支名相同，则可以省略:
git pull <远程主机名> <远程分支名>
# 远程主机名通常为origin
```

#### part 2

##### 查看远程分支

```shell
git branch -r
```

##### 查看分支连接情况

```shell
git branch -vv
# 连接好的分支可以直接使用git push和git pull，而不用带上分支名
```

##### 创建本地分支并连接远程分支

```shell
git checkout -b 分支名 远程分支
# 如:
git checkout -b dev origin/dev
```

##### 已有本地分支连接远程分支

```shell
git branch --set-upstream-to=origin/<branch> 本地分支名
# 在必要时，git命令行会提示的
```

#### part 3

##### 忽略特殊文件

在根目录下编写 `.gitignore`文件，`.gitignore`作用于工作区到暂存区的文件添加

##### 强制添加

```shell
git add -f [file]
# 用于添加被.gitignore忽略的文件
```

##### 检查文件忽略规则

```shell
git check-ignore -v [file]
```

### 5. 配置命令缩写

```shell
git config [--global] alias.缩写 '原命令'
# 如
git config --global alias.lpa 'log --pretty=oneline --abbrev-commit'
# 则 'git lpa' == 'git log --pretty=oneline --abbrev-commit'
```

### 6. 标签管理

##### 创建标签

```shell
git tag 标签名 
# 默认为最新一次commit创建标签
git tag 标签名 [commit id]
# 为commit id创建标签
git tag -a 标签名 -m "描述" [commit id]
```

##### 查看所有标签

```shell
git tag
# 按照字典排序
```

##### 查看标签详细信息

```shell
git show 标签名
# 含描述信息
```

##### 删除标签

```shell
git tag -d 标签名
```

##### 推送标签到远程仓库

```shell
# 推送单个标签
git push origin 标签名
# 推送所有标签
git push origin --tags
```

##### 删除远程仓库的标签

```shell
git tag -d 标签名
git push origin :标签名
```

### 
