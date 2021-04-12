[TOC]

# 申明

内容来自网络，我只是记录一边方便加深记忆，以及之后根据大纲快速查找

# 1. 配置信息

## 1.1 用户信息

`git config  `命令用于设置/查看 配置信息，那么也包括用户信息

### 1.1.1 查看用户信息

* `git config --global user.name` :    //获取当前登录的用户
* `git config --global user.email` :   //获取当前登录用户的邮箱

### 1.1.2 设置/登录用户

* `git config --global user.name` 'uesename' :    //获取当前登录的用户
* `git config --global user.email`'email' :   //获取当前登录用户的邮箱

## 1.2 查看所有配置信息

* `git config --list`

# 2.  获取命令使用帮助

## 2.1 查看所有命令

* `git help`
* `git --help`

## 2.2 查看特定命令

比如查看config相关的命令：`git help config`

# 3. git 仓库

关于git仓库有两种方式

## 3.1 对现有的目录进行 git 管理，叫做初始化仓库

进入需要管理的目录中，并且输入：

`git init`

这个命令就会在你的目录中创建一个.git的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干。 但是，在这个时候，我们仅仅是做了一个初始化的操作，你的目录里的文件还没有被跟踪。

如何跟踪这些文件我会在之后的小结中说明

## 3.2 克隆已存在的仓库

如果你想克隆一份已存在的仓库，那就可以使用：`git clone [url]`

* 比如克隆 github上的项目`git clone https://github.com/xxxx/xxxx`
* 你也可以在克隆的同时自定义本地文件夹名 `git clone https://github.com/xxxx/xxxx consumName`

这里需要说明的是，git支持多种传输协议，不光是这里例子中的https协议

## 3.3 本地仓库中的文件状态

在3.1中我们有提到目录中的我文件还没有跟踪的，相对应的自然有跟踪，所以仓库中的每一个文件的状态不外乎这两种了：

* 已跟踪 ：被纳入版本控制的文件，初次使用 3.2 中的方法克隆的仓库中文件都属于已跟踪文件，并处于未修改状态
* 未跟踪 

已跟踪文件在一段时间之后可能处于：

* 未修改
* 已修改
* 已放入暂存区

自上次提交后做了修改的文件，Git 将它们标记为已修改文件。 我们逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。 

### 3.3.1 检查当前文件状态

`git status`命令，在克隆仓库后立即使用此命令，会看到类似这样的输出： 

```
$ git status
On branch master
nothing to commit, working directory clean
```

这说明你现在的工作目录相当干净。换句话说，所有已跟踪文件在上次提交后都未被更改过。 此外，上面的信息还表明，当前目录下没有出现任何处于未跟踪状态的新文件，否则 Git 会在这里列出来。 最后，该命令还显示了当前所在分支，并告诉你这个分支同远程服务器上对应的分支没有偏离。 现在，分支名是 “master”,这是默认的分支名。 

让我们在项目下创建一个新的 redeme.md 文件。 如果之前并不存在这个文件，使用 `git status` 命令，你将看到一个新的未跟踪文件： 

 ```
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        "redeme.md"

nothing added to commit but untracked files present (use "git add" to track)

 ```

#### 跟踪新文件

**使用  `git add [file/path]` 命令跟踪新的文件**

**`git add` 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。** 

比如：上文中的  redeme.md 文件

输入：`git add redeme.md`，然后输入  `git status`可以看到新的信息

    On branch master
    
    Your branch is up to date with 'origin/master'.
    
    Changes to be committed:
    
      (use "git reset HEAD <file>..." to unstage)
        new file:   "redeme.md"

只要在 `Changes to be committed` 这行下面的，就说明是已暂存状态。 如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中。 

#### 状态预览

```
git status
```

#### 状态简览

```
git status -s   
```

#### 暂存已修改文件

现在我们来修改一个已被跟踪的文件。 如果你修改了一个之前的 `redeme.md` 的已被跟踪的文件，然后运行 `git status` 命令，会看到下面内容：

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   "redeme.md"

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   "redeme.md"
```

文件 `redeme.md` 出现在 `Changes not staged for commit` 这行下面，说明已跟踪文件的内容发生了变化，但还没有将最新的内容放到暂存区，出现在`Changes to be committed` 下面，是因为那里存放了之前未提交的版本数据。 要暂存这次更新，需要运行 **`git add` 命令。 这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“添加内容到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。** 现在让我们运行 `git add` 将"CONTRIBUTING.md"放到暂存区，然后再看看 `git status` 的输出：

```
$ git add redeme.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   "redeme.md"
```

现在文件已暂存，下次提交时就会将修改的一并记录到仓库。

#### 忽略文件

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 

例子：

```
cat .gitignore
*.[oa]
*~
```

第一行告诉 Git 忽略所有以 `.o` 或 `.a` 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的。 第二行告诉 Git 忽略所有以波浪符（~）结尾的文件，许多文本编辑软件（比如 Emacs）都用这样的文件名保存副本。 此外，你可能还需要忽略 log，tmp 或者 pid 目录，以及自动生成的文档等等。 要养成一开始就设置好 .gitignore 文件的习惯，以免将来误提交这类无用的文件。 

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `＃` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配。
- 匹配模式可以以（`/`）开头防止递归。
- 匹配模式可以以（`/`）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（`*`）匹配零个或多个任意字符；`[abc]`匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（`?`）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。 使用两个星号（`*`) 表示匹配任意中间目录，比如`a/**/z` 可以匹配 `a/z`, `a/b/z` 或 `a/b/c/z`等。

我们再看一个 .gitignore 文件的例子：

```
# no .a files
# 忽略任何以.a结尾的文件
*.a

# but do track lib.a, even though you're ignoring .a files above
# 排除/除...以外，也就是说上面忽略所有.a的文件但是除了lib.a以外
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
# 忽略 当前目录中的 /TODO 文件，不包括子目录中的 /TODO 文件
/TODO

# ignore all files in the build/ directory
# 忽略这个 build 目录下的所有文件
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
# 忽略 doc/ 目录下的 任何.txt文件，不包括子目录
doc/*.txt

# ignore all .pdf files in the doc/ directory
# 忽略 doc/ 目录下的 任何.pdf文件，包括子目录
doc/**/*.pdf
```

## 3.4 比较差异

### 比较文件暂存区和工作区的差异

一个修改过的文件，通过 `git add`  命令暂存之后，如果再次修改这时候可以通过 `git diff`命令来查看修改的内容

**这个命令的退出呢，按 q 键即可**

### 比较的是暂存区和历史区的差异 

使用 `git diff --cached`( 这两个都可以--staged 和 --cached 是同义词) 

### 比较的是历史区和工作区的差异（修改） 

`git diff master `

## 3.5 提交更新

**注意：我们这里的提交更新，提交到的都是本地的仓库中，如果需要提交到远程仓库，请看后续章节**

提交更新前请检查你所有需要提交的文件都暂存过了（调用过 git add 命令）。

提交更新使用 `git commit`命令，他将会打开一个编辑器，需要你输入提交的说明信息

你也可以使用 `git commit -m "commitChanged"` 来将提交的说明信息与命令放在同一行 

### 跳过使用使用繁琐的 `git add`命令的提交方式

`git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤 

`git commit -a -m "commitChanged"`

## 3.6 覆盖本地

`git checkout filename`

**后续会继续更新内容** 

## 3.7 查看提交历史

`git log`

一个常用的选项是 `-p`，用来显示每次提交的内容差异。 你也可以加上 `-2` 来仅显示最近两次提交

更多请查看https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2

## 3.8 撤销

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令尝试重新提交：

```console
$ git commit --amend
```

这个命令会将暂存区中的文件提交。 如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令），那么快照会保持不变，而你所修改的只是提交信息。

文本编辑器启动后，可以看到之前的提交信息。 编辑后保存会覆盖原来的提交信息。

例如，你提交后发现忘记了暂存某些需要的修改，可以像下面这样操作：

```console
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

最终你只会有一个提交 - 第二次提交将代替第一次提交的结果。

### 取消暂存的文件

`git reset HEAD CONTRIBUTING.md`

# 2. 远程仓库的使用

更多使用查看https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8

## 2.1 查看远程仓库

`git remote`: 我们可以查看远程仓库,如果你的远程仓库有多个，那么会给你全部列出来

```
$ git remote
origin
```

使用 -v 选项可以看到其对应的url信息

```
$ git remote -v
origin  git@git.dev.sh.ctripcorp.com:xxxxxxx/xxxxxxx.git (fetch)
origin  git@git.dev.sh.ctripcorp.com:xxxxxxx/xxxxxxx.git (push)
```

## 2.2 增加远程仓库

`git remote add <shortname> <url>`

## 2.3 从远程仓库中抓取与拉取

```console
$ git fetch [remote-name]
```

这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。**它并不会自动合并或修改你当前的工作。**

**注意 ： 这个命令一般很少使用，我们一般都拉去对应的介个分支，不会全部拉下来**，我们一般使用下面的方式

### 拉去远程仓库的更新

`git pull origin [分支名]` 

### 提交到远程仓库

`git push origin [分支名]` 命令，如果相同文件在远程仓库有修改，提交将失败

比如提交到master分支

`git push origin master` 

# 3 分支

[分支简介](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%AE%80%E4%BB%8B)

## 3.1 分支创建

`git branch [分支名]`

### 新建并切换

`git branch -b [分支名]`他是下面两条命令的简写

```console
$ git branch [分支名]
$ git checkout [分支名]
```

## 3.2 分支查看

* `git branch`  查看本地分支
* `git branch -r`  查看远程分支
* `git branch -va`  查看本地和远程分支

## 3.3  分支切换

```console
git checkout 分支名
```

下面这个命令是拉取远程分支代码以及创建本地分支指向这个远程分支，并且切换到这个新建的分支

```
git checkout -b [新建本地分支名] origin/[对应的远程分支]
```

## 3.4 合并分支

如果你想更好了理解分支合并请看https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6

首先切换到你需要合并的分支上，比如：

```console
git checkout master
```

然后合并分支到这个master,这个hotfix是需要合并到master的分支

```console
git merge hotfix
```

## 3.5 遇到冲突时的分支合并

合并操作还是3.4的方法，此时 Git 做了合并，但是没有自动地创建一个新的合并提交。 Git 会暂停下来，等待你去解决合并产生的冲突。 你可以在合并冲突后的任意时刻使用 `git status` 命令来查看那些因包含合并冲突而处于未合并（unmerged）状态的文件：

```console
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:      index.html

no changes added to commit (use "git add" and/or "git commit -a")
```

任何因包含合并冲突而有待解决的文件，都会以未合并状态标识出来。 Git 会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的文件然后手动解决冲突。 出现冲突的文件会包含一些特殊区段，看起来像下面这个样子：

```html
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

这表示 `HEAD` 所指示的版本（也就是你的 `master` 分支所在的位置，因为你在运行 merge 命令的时候已经检出到了这个分支）在这个区段的上半部分（`=======` 的上半部分），而 `iss53` 分支所指示的版本在 `=======` 的下半部分。 为了解决冲突，你必须选择使用由 `=======` 分割的两部分中的一个，或者你也可以自行合并这些内容。 

**在你解决了所有文件里的冲突之后，对每个文件使用 `git add` 命令来将其标记为冲突已解决。 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。**

## 3.6 删除分支

```console
git branch -d hotfix
```

## 3.7 分支管理

`-v` 可以查看分支最后一次提交的信息

## 3.8 用一个分支完全覆盖另一个分支

当前分支是maser分支，我想讲paytest分支上的代码完全覆盖master分支，首先切换到master分支。

```
git reset --hard origin/paytest
```

然后将本地分支强行推到远程分支。

```
git push -f
```

## 3.9 本地分支关联远程分支

远程分支存在的情况

```
git branch --set-upstream-to origin/远程分支
```

远程分支不存在的情况

```
git push origin newdev:dev
```

可以将本地的 newdev 分支 和  origin/dev 分支关联（origin/dev不存在会创建一个）

