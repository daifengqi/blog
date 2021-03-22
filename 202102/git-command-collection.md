# 常用 git 指令收集

# Git Command Collection

## 2021 年 2 月 28 日

## Feb 28 2021

---

**克隆仓库到本地**

```shell
git clone https://github.com/<path>.git
```

附上一个[Chrome 拓展](https://chrome.google.com/webstore/detail/gitzip-for-github/ffabmkklhbepgcgfonabamgnfafbdlkn?hl=en)，可以指定 Github 的单个文件夹下载，可以用于克隆一些大型项目的部分文件。

**查看仓库绑定关系**

```shell
git remote -v
```

一般说来有两种，origin 表示直接绑定的仓库，一般指向自己的仓库；upstream 表示 fork 的源仓库。

**添加、修改与提交**

```shell
git add .
git commit -m "commit message"
git push origin main
```

代码提交到 origin 后，就可以提交 pr 请求，从而向源仓库贡献代码。

**添加源仓库**

```shell
git remote add origin https://github.com/<path>.git
git remote add upstream https://github.com/<path>.git
```

一般用来绑定自己 fork 的仓库和源仓库。

**从源仓库取文件到本地、合并**

```shell
git fetch upstream main
git checkout main
git merge upstream/main
```

这个操作会去取 upstream 仓库的最新内容，然后合并到本地仓库。

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
