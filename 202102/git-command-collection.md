# 常用 git 指令收集

# Git Command Collection

## 2021 年 2 月 28 日

## Feb 28 2021

---

这篇文章带有方括号`<>`的变量为指代名称，用户需要根据上下文修改为相应的内容。

### 配置相关

**首次配置用户名（user.name）和邮箱（email）**，以名字为Cedric Qi和邮箱为cescdf@me.com为例

```shell
git config --global user.name "Cedric Qi"
git config --global user.email cescdf@me.com
```

**查看目前的所有配置值**

```shell
git config --list
```

**查看单个配置的值**，比如要看user.name

```shell
git config user.name
```

### 开发相关

**克隆仓库到本地**

```shell
git clone https://github.com/<path>.git
```

附上一个[Chrome 拓展](https://chrome.google.com/webstore/detail/gitzip-for-github/ffabmkklhbepgcgfonabamgnfafbdlkn?hl=en)，可以指定 Github 的单个文件夹下载，可以用于克隆一些大型项目的部分文件。

对于使用npm的用户，也可以通过[degit](https://github.com/Rich-Harris/degit)快速克隆部分文件。

**查看仓库状态**

```shell
git status
```

显示工作目录和临时区域的状态，查看到哪些更改已经提交，哪些文件没有被跟踪。

**查看仓库绑定关系**

```shell
git remote -v
```

一般说来有两种，origin 表示直接绑定的仓库，一般指向自己的仓库；upstream 表示 fork 的源仓库。这两个单词可以看作一种标志。

以上两个命令可用来快速判断当前路径是否是一个git仓库。

**添加、修改与提交**

```shell
git add .
git commit -m "commit message"
git push origin main
```

`main`表示分支的名称，一般叫做master或者main的是作为主分支。代码提交到 origin 后，就可以提交 pr 请求，从而向上游仓库贡献代码。

第一行的点`.`表示匹配所有文件，如果只是要提交修改单个文件，比如某文件叫`test.py`，那么只需要把指令修改为`git add test.py`。

"commit message"是一种留言，用于简短地说明本次做了什么修改，作为记录。

**撤销上次提交**

```shell
git reset HEAD~
```

这是一个比较复杂的问题，关于撤销提交的讨论请参考这个[stackoverflow](https://stackoverflow.com/questions/927358/how-do-i-undo-the-most-recent-local-commits-in-git)。

**本地绑定远程的仓库和上游仓库**

```shell
git remote add origin https://github.com/<path>.git
git remote add upstream https://github.com/<path>.git
```

一般用来绑定自己 fork 的仓库和源仓库。origin是自己的远程仓库，upstream是上游仓库。

**本地删除绑定仓库**

```shell
git remote remove origin https://github.com/<path>.git
git remote remove upstream https://github.com/<path>.git
```

与上面的操作相反，这里在词法上也仅仅是把`add`修改为了`remove`

**从上游仓库取文件到本地、合并**

```shell
git fetch upstream main
git checkout main
git merge upstream/main
```

这个操作会去取 upstream 仓库的最新内容，然后合并到本地仓库。

**从远处仓库取文件到本地**

```shell
git pull origin main
```

这里pull可以理解为push的反向操作，push是从本地“推”到远程服务器，而pull是从远程服务器“拉”到本地。

**创建、查看、切换、合并分支**

```shell
# 创建
git branch <branch-name>
# 查看
git branch -l
# 切换
git checkout <branch-name>

# 合并
git checkout master
git merge <branch-name>
```

**查看提交日志、回退**

```shell
git log
git show <id>
git reset <id>
```

在使用`git log`查看回滚日志时，若想退出，需按 q 键。
