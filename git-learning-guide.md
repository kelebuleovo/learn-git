# Git 命令行学习手册

> 写给你的一本"边做边学"的 Git 入门手册。
>
> - 仓库：`~/Desktop/learn-git`（远程 `git@github.com:kelebuleovo/learn-git`）
> - 环境：Windows 10 + Git 2.56.1 + Git Bash
> - 你已经配好了 `user.name = kelebuleovo`、`user.email = kelebuleovo@gmail.com`，并且用 SSH 连上了远程仓库
> - 学习顺序：先把**本地操作**练熟（第 0～8 章），远程推送放在最后（附录 A）

---

## 写在前面：怎么用这份手册

1. **不要只读，要敲。** 读完一段就打开 Git Bash 敲一遍，看到输出再往下读。眼睛会骗人，手指不会。
2. **先猜，再对答案。** 每章有"动手做"和小测。答案用折叠块藏起来了，先自己想一想，再点开。
3. **出错是学习的一部分。** 这份手册里的操作都在你自己的练习仓库里，弄坏了也能重来。每次敲完 `git status`，先读一遍 Git 在说什么——它其实一直在给你提示。
4. **所有命令默认在 Git Bash 里执行**（不是 cmd，也不是 PowerShell），这样命令能和手册完全一致。PowerShell 的差异见附录 B。

开始之前，先确认三件事：

```bash
cd ~/Desktop/learn-git    # 进入仓库目录
pwd                       # 应该显示 /c/Users/Administrator/Desktop/learn-git
git --version             # 应该显示 git version 2.56.1.windows.1
```

如果 `cd` 失败，先用 `ls ~/Desktop` 看看文件夹在不在。

### 目录

- [第 0 章 先认识你手里的仓库](#ch0)
- [第 1 章 把抽象概念讲清楚](#ch1)
- [第 2 章 第一次用命令行提交](#ch2)
- [第 3 章 读懂 status 和 diff](#ch3)
- [第 4 章 撤销与修正：本地安全操作](#ch4)
- [第 5 章 分支：平行时间线](#ch5)
- [第 6 章 .gitignore：告诉 Git 哪些东西不用管](#ch6)
- [第 7 章 LICENSE：开源许可证](#ch7)
- [第 8 章 常见报错翻译器](#ch8)
- [第 9 章 自测清单与综合练习](#ch9)
- [附录 A 远程仓库：下一步](#appendix-a)
- [附录 B PowerShell 差异与 Windows 小贴士](#appendix-b)
- [附录 C 命令速查表](#appendix-c)
- [附录 D 术语中英对照](#appendix-d)
- [附录 E 学习节奏建议](#appendix-e)

---

<a id="ch0"></a>
## 第 0 章 先认识你手里的仓库

### 0.1 你已经有的东西

我替你先看了一眼这个仓库。它不是一个空文件夹，而是已经是一个**完整的 Git 仓库**了：

| 你拥有的 | 说明 |
| --- | --- |
| `.git/` 目录 | Git 的"数据库 + 设置"，所有历史记录都在这里 |
| `README.md` | 只有一行 `# learn-git`（注意：**文件末尾没有换行符**） |
| `LICENSE` | MIT 许可证，版权行是 `Copyright (c) 2026 kelebuleovo` |
| `.gitignore` | GitHub 的 Python 模板，已经忽略 `.env`、`*.log`、`__pycache__/` 等 |
| 第一个提交 `a3164a7` | 提交信息是 `Initial commit`，`main` 分支跟踪 `origin/main` |
| 远程 `origin` | `git@github.com:kelebuleovo/learn-git`，SSH 方式 |

> **老师的话**：你可能会问——"我想学用命令行创建 README、许可证、.gitignore，可它们已经存在了怎么办？"  
> 答案是：**先读懂已有的，再用命令行去修改、扩展它们**。这比机械地"从零创建"更有价值，因为真实工作中你 90% 的时间都在改已经存在的仓库。当然，每类文件我都会讲清楚"如果从零创建该怎么做"。

> **注意**：这份手册 `git-learning-guide.md` 是我刚放进来的，Git 现在还不认识它——它是一个**未跟踪文件**。这正好是你遇到的第一个真实状态。它会在 `git status` 里一直出现，直到你决定提交它。先让它待着，别急。

### 0.2 三个区域：一张图记住 Git 的核心

初学者最容易晕的地方，是 Git 有两处"中转站"。用"打包寄快递"来理解：

```
 ┌───────────────┐   git add    ┌────────────────┐   git commit   ┌───────────────┐
 │   工作区       │ ───────────> │    暂存区       │ ────────────>  │   本地仓库     │
 │ Working Tree  │ <─────────── │ Index / Stage  │ <────────────  │   (.git)      │
 │ 你眼前能改的文件│ git restore  │  "打包台"       │ restore --staged│  "存档柜"      │
 └───────────────┘              └────────────────┘                └───────┬───────┘
                                                                          │ git push
                                                                          v
                                                                  ┌───────────────┐
                                                                  │   远程仓库     │
                                                                  │ origin (GitHub)│
                                                                  └───────────────┘
```

- **工作区**：你在文件管理器里看到的、能用编辑器改的那些文件。
- **暂存区（Index / Stage）**：一个"打包台"。你把想放进这次提交的改动挑出来放上去。它不显眼，但它是 Git 设计里最关键的一步。
- **本地仓库**：`.git` 目录里保存的完整历史。每次 `git commit` 就是给项目拍一张快照，存进档案柜。
- **远程仓库**：GitHub 上的副本。`push` 是把本地提交送上去，`pull` 是把远程的拿下来。

`git status` 永远在告诉你：**工作区、暂存区、本地仓库这三者现在哪里不一致**。

### 0.3 `.git` 里到底有什么

`ls .git` 会看到这些：

| 名字 | 作用（先有个印象，不用背） |
| --- | --- |
| `HEAD` | 一个文本文件，写着"你现在在哪个分支"，内容通常是 `ref: refs/heads/main` |
| `config` | 这个仓库自己的配置（远程地址、分支设置等） |
| `refs/heads/` | 本地分支。每个分支就是一个小文件，里面存一个提交哈希 |
| `refs/remotes/` | 远程分支在你本地的"镜像" |
| `objects/` | 所有文件内容和提交对象，按哈希存放 |
| `index` | 暂存区的实际数据文件 |
| `logs/` | 引用变化日志，`git reflog` 读的就是它 |
| `hooks/` | 自动化脚本钩子（现在不用管） |

**原则：不要手动改 `.git` 里的文件。** 看可以，改要用 Git 命令。你可以把 `.git` 理解成"银行的金库"——你可以隔着玻璃看，但别自己伸手进去。

### 动手做 0：第一次观察

依次执行，并回答后面的问题：

```bash
git status
git log --oneline --graph --all --decorate
cat .git/HEAD
cat .git/refs/heads/main
git branch -vv
git remote -v
```

你会看到类似这样的输出（哈希和你的可能一致）：

```text
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	git-learning-guide.md

nothing added to commit but untracked files present (use "git add" to track)
```

```text
$ git log --oneline --graph --all --decorate
* a3164a7 (HEAD -> main, origin/main, origin/HEAD) Initial commit
```

```text
$ cat .git/HEAD
ref: refs/heads/main
```

```text
$ cat .git/refs/heads/main
a3164a70e4ef11542eecafa4facec34dcab0aaaa
```

**想一想**：

1. `HEAD -> main` 和 `origin/main` 分别是什么意思？
2. 为什么 `git status` 说 "Your branch is up to date with 'origin/main'"，却又列出一个未跟踪文件？
3. `.git/refs/heads/main` 里那串 40 位字符是什么？它和 `git log` 里的 `a3164a7` 是什么关系？
4. `origin` 是什么？`origin/main` 是远程仓库上真实存在的分支吗？

<details>
<summary>点我看参考答案</summary>

1. `HEAD -> main` 表示你当前站在 `main` 分支上；`origin/main` 是"你上次和远程通信时，远程 main 所在的位置"的本地记录（远程跟踪分支）。
2. "up to date" 只比较**已经提交的历史**：你的 `main` 和 `origin/main` 指向同一个提交。未跟踪文件还没进入历史，所以不影响这个判断。
3. 它是提交的完整哈希（40 位十六进制）。`a3164a7` 只是它的前 7 位缩写，日常使用足够区分。
4. `origin` 只是远程仓库地址的**别名**（默认叫 origin，可以改）。`origin/main` 不是你本地的分支，而是 Git 帮你记的"远程 main 的镜像"，是只读的；`git fetch` 会更新它。

</details>

> **小贴士**：你的全局配置里 `init.defaultBranch` 还是 `master`（Git for Windows 的默认值）。这个仓库已经是 `main` 了，不受影响；但以后新建仓库时你大概也想用 `main`。可以现在就改：
>
> ```bash
> git config --global init.defaultBranch main
> ```
>
> 这条命令以后 `git init` 新建仓库时默认用 `main`。

---

<a id="ch1"></a>
## 第 1 章 把抽象概念讲清楚

这一章没有操作，但请认真读。概念不清，命令敲得再熟也只是背口诀。

### 1.1 提交是"快照"，不是"改了哪几行"

大多数人以为 Git 每次只记录"增删了哪几行"。更准确的理解是：**每次提交，Git 都会给整个项目拍一张完整的快照**（逻辑上是完整快照；实现上会复用没变的文件，所以并不臃肿）。

- `git diff` 显示的"增删行"，是 Git 在**临时比较**两张快照，不是它只存了差异。
- 所以你可以随时切到任何一个提交，看到那个时刻的**完整项目状态**。

用游戏存档类比：`git commit` 就是"保存游戏"，每一档都是完整的世界状态；`git diff` 是"对比两个存档的差异"。

### 1.2 哈希：提交的身份证号

每个提交都有一个 40 位的十六进制编号，例如 `a3164a70e4ef...`。它由提交的内容（文件树、作者、时间、说明、父提交等）计算出来：

- 同样的内容 → 同样的编号；
- 任何一点内容变了（哪怕只改提交说明）→ 编号完全变样。

所以哈希不是随机数，而是"内容的指纹"。日常你只需要用前 7 位，例如 `a3164a7`。

### 1.3 HEAD：你现在站在哪

`HEAD` 是 Git 的"当前指针"，指向你**正在查看/操作的分支**。`cat .git/HEAD` 显示：

```text
ref: refs/heads/main
```

意思是"HEAD 指向 main 分支"。而 `main` 这个分支本身又指向某个提交。所以：

```
HEAD → main → a3164a7（最新提交）
```

当你切换分支时，HEAD 跟着换指向；工作区的文件也会随之变成那个分支的状态。

> 后面你会听到 "detached HEAD"（游离头指针）：HEAD 不指向任何分支，而是直接指向某个提交。先记个名字，第 8 章会解释。

### 1.4 分支：贴在提交上的便利贴

**分支不是文件夹的副本，也不是复制一份代码。** 分支只是一个"可移动的指针"，指向某个提交。创建分支基本上零成本——只是在 `.git/refs/heads/` 下多一个小文件。

```
main    : A ─── B ─── C
                      ↑
                    HEAD（你在这）

dev     : A ─── B ─── D
                      ↑
                    dev 指针
```

`main` 和 `dev` 共享 A、B 两个提交，只是各自指向不同的最新提交。你在 `dev` 上提交，只有 `dev` 指针前移，`main` 不动。这就是"平行时间线"。

### 1.5 暂存区：为什么非要"多一步"

新手常问：为什么改完不能直接提交，非要先 `git add`？

因为**一次提交应该是一组有意义的改动**。暂存区让你能：

- 只提交一部分文件，把半成品留在工作区；
- 把一个大改动拆成几个逻辑清晰的提交；
- 提交前再检查一遍"这次到底装了什么"（`git diff --staged`）。

它不是官僚流程，而是"打包前先挑货"的桌子。习惯之后，你会感谢这一步。

### 1.6 本地 vs 远程：两张网

- **本地仓库**：你电脑上的 `.git`，完整历史都在这里。**没有网络也能提交、开分支、合并。**
- **远程仓库**：GitHub 上的副本，用于备份和协作。`git push` 上传本地提交；`git pull` 下载远程提交。
- 远程不是"唯一的真相"。本地和远程可以暂时不一致：你提交了但没 push，本地就"领先"远程几个提交。`git status` 会告诉你领先/落后多少。

### 小测（先别看答案）

1. `git add` 把文件放进了哪里？`git commit` 之后文件又在哪里？
2. 创建一个新分支，会复制整个项目的文件吗？
3. 改了提交说明，提交的哈希会变吗？
4. `HEAD`、`main`、`a3164a7` 三者的指向关系是什么？
5. 没有网络时，还能 `git commit` 吗？

<details>
<summary>点我看答案</summary>

1. `git add` 把**当前内容**放进暂存区；`git commit` 把暂存区的内容打包成一个提交，存入本地仓库（`.git`）。
2. 不会。分支只是一个指向提交的指针，创建几乎零成本。
3. 会。提交说明是提交内容的一部分，改了就会得到全新的哈希。
4. `HEAD → main → a3164a7`。HEAD 指向当前分支，分支指向最新提交。
5. 能。提交是纯本地操作；只有 `push` / `pull` / `fetch` 需要网络。

</details>

---

<a id="ch2"></a>
## 第 2 章 第一次用命令行提交

目标：用命令行修改 `README.md`，完成一次属于你的提交。

### 2.1 先看一眼现状

```bash
git status
```

你应该看到 `Untracked files: git-learning-guide.md`。这就是"未跟踪"：文件存在于工作区，但 Git 还没把它纳入版本管理。

同时注意 README 的一个小细节：它只有 11 个字节，**末尾没有换行符**。如果你直接用 `echo "..." >> README.md` 追加，新内容会紧贴在第一行后面，变成 `# learn-git## 学习记录`。所以我们用 `printf`，先补上换行：

```bash
printf '\n\n## 学习记录\n\n- 2026-11-24：完成第一次命令行提交。\n' >> README.md
```

（开头两个 `\n`：第一个补上原文件缺的换行，第二个留出空行。）

> 如果你更习惯用编辑器，也可以直接打开 README.md 手动加内容——效果一样。命令行的意义在于"你能在任何没有图形界面的机器上做同样的事"。

### 2.2 看看工作区发生了什么

```bash
git status
```

输出会变成：

```text
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	git-learning-guide.md

no changes added to commit (use "git add" and/or "git commit -a")
```

读法：

- `modified: README.md`：文件被改了，但还**没放进暂存区**；
- `Untracked files`：新文件，Git 根本不认识；
- 最后一行是 Git 的建议：用 `git add` 把改动放进去。

### 2.3 `git diff`：看清改了哪些行

```bash
git diff
```

你会看到类似：

```diff
diff --git a/README.md b/README.md
index abc1234..def5678 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,5 @@
-# learn-git
\ No newline at end of file
+# learn-git
+
+## 学习记录
+
+- 2026-11-24：完成第一次命令行提交。
```

逐段读：

- `--- a/README.md` / `+++ b/README.md`：旧版本 / 新版本；
- `@@ -1 +1,5 @@`：旧文件第 1 行，新文件第 1～5 行；
- `-` 开头的行是删掉的，`+` 开头的是新增的，没符号的是上下文；
- `\ No newline at end of file` 表示原文件末尾没有换行——这正是我们刚才处理掉的东西。

> **重要**：`git diff` 默认只显示**工作区 vs 暂存区**的差异。所以你现在看到的是"还没打包的改动"。

### 2.4 `git add`：把改动放到打包台

```bash
git add README.md
```

只添加 README，不添加那份手册。再次 `git status`：

```text
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   README.md

Untracked files:
	git-learning-guide.md
```

现在 README 的改动在暂存区了。看一下暂存区里的内容：

```bash
git diff --staged
```

`--staged` 和 `--cached` 是同一个意思：**暂存区 vs 最后一次提交**。此时 `git diff`（工作区 vs 暂存区）应该是空的，因为你刚把工作区的状态原样放进了暂存区。

> **关于 `git add .`**：它会添加当前目录下所有改动和所有未忽略的新文件——包括这份手册。新手建议先显式写出文件名，确认自己到底在打包什么；熟练后再用 `.` 图省事。

### 2.5 `git commit`：封箱存档

```bash
git commit -m "docs: 在 README 添加学习记录"
```

- `-m` 后面是提交说明。**新手请一律先加 `-m`**，否则 Git 会打开一个编辑器（通常是 vim），很多人会卡在里面。
- 真的不小心进去了：按 `Esc`，输入 `:wq` 回车保存退出；想放弃就输入 `:q!` 回车。

提交成功后，Git 会打印类似：

```text
[main 5c1d2e3] docs: 在 README 添加学习记录
 1 file changed, 5 insertions(+), 1 deletion(-)
```

> 如果你看到 `warning: LF will be replaced by CRLF ...`，这是 Windows 换行符转换（`core.autocrlf=true`）的正常提示，不是错误，可以忽略。

然后：

```bash
git status
```

```text
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

Untracked files:
	git-learning-guide.md

nothing added to commit but untracked files present (use "git add" to track)
```

"ahead of 'origin/main' by 1 commit" 的意思是：本地比远程多一个提交。**这是正常的**，本地学习阶段不需要 push。

### 2.6 确认提交里有什么

```bash
git log --oneline --graph --decorate --all
git show --stat HEAD
git show HEAD:README.md
```

- `git log ... --all`：画出提交图，显示所有分支标签；
- `git show --stat HEAD`：显示最新提交的摘要和改了哪些文件；
- `git show HEAD:README.md`：直接查看"最新提交里"的 README 内容（不读工作区）。

输出大概长这样：

```text
* 5c1d2e3 (HEAD -> main) docs: 在 README 添加学习记录
* a3164a7 (origin/main, origin/HEAD) Initial commit
```

注意 `origin/main` 还停在初始提交——因为你还没 push。**本地历史已经向前走了，远程没有。**

### 2.7 提交说明怎么写

一条好的提交说明，是写给**三个月后的自己和同事**看的。建议：

- 用**动词开头的祈使句**：`添加...`、`修复...`、`重构...`，而不是"我添加了..."；
- 一句话说清**做了什么**，必要时在第二段说**为什么**；
- 前缀可选但好用：`feat:`（功能）、`fix:`（修复）、`docs:`（文档）、`chore:`（杂务）、`refactor:`（重构）；
- 别写 `update`、`修改`、`提交一下` 这种没有信息量的说明。

对比：

| 不推荐 | 推荐 |
| --- | --- |
| `update` | `docs: 在 README 添加学习记录` |
| `改了一下` | `fix: 修正 README 中的错误命令` |
| `提交` | `feat: 添加许可证说明链接` |

### 动手做 2：亲自走一遍

1. 按上面的步骤修改 README、`git diff`、`git add`、`git diff --staged`、`git commit`。
2. 提交后运行 `git log --oneline --graph --decorate --all`，确认新提交在 `main` 上。
3. 运行 `git status`，确认只剩那份手册是 untracked 的。

<details>
<summary>如果你想知道"从零创建 README"怎么做</summary>

在一个**空目录**里（不要在 learn-git 里做，避免破坏现有文件）：

```bash
mkdir -p ~/Desktop/git-sandbox && cd ~/Desktop/git-sandbox
git init -b main
printf '# 我的新项目\n\n这是 README。\n' > README.md
git add README.md
git commit -m "docs: 添加 README"
```

`git init -b main` 里的 `-b` 指定初始分支名（Git 2.28+ 支持）。

</details>

---

<a id="ch3"></a>
## 第 3 章 读懂 status 和 diff

### 3.1 `git status` 的三种身份

Git 眼里的文件只有几种状态：

| 状态 | 含义 | 怎么变成这样 |
| --- | --- | --- |
| **untracked**（未跟踪） | 新文件，Git 没管过 | 新建文件 |
| **modified**（已修改） | 跟踪过，工作区内容与暂存区不同 | 编辑已跟踪文件 |
| **staged**（已暂存） | 改动已放进暂存区，等待提交 | `git add` |
| **unmodified**（未修改） | 工作区、暂存区、最新提交三者一致 | 提交后 |

### 3.2 `--short`：两列密码

```bash
git status --short
```

输出类似：

```text
 M README.md
?? git-learning-guide.md
AM practice-note.txt
```

两列的含义是：

- **第一列（左）**：暂存区 vs 最新提交（HEAD）的差异；
- **第二列（右）**：工作区 vs 暂存区的差异。

常见组合：

| 代码 | 含义 |
| --- | --- |
| `??` | 未跟踪 |
| ` M` | 工作区改了，没暂存 |
| `M ` | 改了并已暂存 |
| `MM` | 暂存后又改了（暂存的是一版，工作区是另一版） |
| `A ` | 新文件已暂存 |
| `AM` | 新文件已暂存，之后又改了 |
| `D ` | 已暂存删除 |
| `R ` | 重命名（已暂存） |

> `AM` 这个状态很重要，第 3.4 节会专门演示它带来的"坑"。

### 3.3 diff 的三种问法

| 命令 | 比较的是 | 典型用途 |
| --- | --- | --- |
| `git diff` | 工作区 ↔ 暂存区 | 看"还没打包"的改动 |
| `git diff --staged` | 暂存区 ↔ 最新提交 | 看"这次要提交什么" |
| `git diff HEAD` | 工作区 + 暂存区 ↔ 最新提交 | 看"相对上次提交的总变化" |

另外：

```bash
git diff --stat              # 只显示改了哪些文件、增减行数
git diff -- README.md        # 只看某个文件
git diff <提交1> <提交2>     # 比较两个提交
```

### 3.4 暂存区的经典陷阱

这是新手最容易困惑的地方。跟着做：

```bash
printf '第一版\n' > practice-note.txt
git add practice-note.txt
printf '第二版\n' >> practice-note.txt
git status --short
```

你会看到：

```text
AM practice-note.txt
```

那意味着：暂存区里是"第一版"，工作区里已经是"第一版 + 第二版"。现在提交：

```bash
git commit -m "practice: 提交 note 的第一版"
git show HEAD:practice-note.txt
```

输出只有 `第一版`。**因为 `git add` 存的是那一刻的内容，不是"这个文件以后的所有变化"。**

要提交第二版，需要再 `git add` 一次：

```bash
git add practice-note.txt
git commit -m "practice: 补充 note 第二版"
```

> **记住这句话**：`git add` 是"拍照"，不是"开始跟踪这个文件的未来"。

### 动手做 3：观察两种 diff

1. 再往 `practice-note.txt` 追加一行，先不要 `git add`。
2. 运行 `git diff` 和 `git diff --staged`，分别看到什么？
3. `git add practice-note.txt` 后再运行这两条命令，输出如何互换？
4. 用 `git status --short` 验证状态从 ` M` 变成 `M `。
5. 最后把这次的练习提交掉，让工作区回到干净状态：

   ```bash
   git commit -m "practice: 记录第三行"
   ```

<details>
<summary>参考答案</summary>

- 未 add 时：`git diff` 显示新追加的那行；`git diff --staged` 为空（暂存区和 HEAD 一致）。
- add 之后：`git diff` 为空（工作区和暂存区一致）；`git diff --staged` 显示新追加的那行。
- 状态：` M`（工作区改了）→ `M `（已暂存）。

</details>

### 3.5 查看历史的常用姿势

```bash
git log --oneline                      # 一行一个提交
git log --oneline --graph --decorate   # 带分支图
git log --oneline --graph --all --decorate   # 所有分支
git log --stat                         # 每个提交改了哪些文件
git log -p -2                          # 最近两个提交的具体差异
git log -- README.md                   # 只看某个文件的历史
git show <哈希>                        # 看某个提交的完整内容
```

`git log` 默认打开分页器（less）。操作：空格翻页，`b` 上一页，`q` 退出。新手最容易以为"卡住了"，其实是在等你看。

---

<a id="ch4"></a>
## 第 4 章 撤销与修正：本地安全操作

这一章教你"改错了怎么退回去"。Git 的好处是：**只要提交过，几乎都能找回来**；但**没提交的改动**，有些命令会直接丢掉，所以每次动手前先 `git diff` 看一眼。

### 4.1 场景一：改了文件，还没 add，想反悔

```bash
printf '\n这一行不想要\n' >> README.md
git diff                      # 确认确实是不想要的内容
git restore README.md
git status                    # 工作区恢复干净
```

`git restore <文件>` 用**暂存区的内容**覆盖工作区。等价的老命令是 `git checkout -- <文件>`，但 `restore` 更清楚、更安全。

> **警告**：`git restore` 会**永久丢弃**工作区里未提交的改动，没有 reflog 可救。先 `git diff` 确认，再执行。

### 4.2 场景二：add 了，但不想提交

```bash
printf '\n先暂存再反悔\n' >> README.md
git add README.md
git status                    # Changes to be committed
git restore --staged README.md
git status                    # 回到 Changes not staged
```

`git restore --staged <文件>` 只把文件从暂存区拿下来，**工作区的改动还在**。等价老命令：`git reset HEAD <文件>`。

如果连工作区改动也不要了，再执行一次 `git restore README.md`。

### 4.3 场景三：刚提交，发现说明写错/漏了文件

**只改说明**：

```bash
git commit --amend -m "docs: 修正后的提交说明"
```

**漏了文件**：

```bash
git add 漏掉的文件
git commit --amend --no-edit
```

`--amend` 不是"新加一个提交"，而是**用新的内容替换上一个提交**。所以哈希会变。

> **重要边界**：`--amend` 只适合**还没 push** 的提交。已经 push 的提交再 amend，本地和远程的历史就分叉了，处理起来很麻烦。已经 push 的提交要改，用后面的 `revert`，或者和团队商量后再 force push（新手先别碰）。

### 4.4 场景四：撤销一个已经提交的改动

```bash
git revert HEAD
```

- `revert` 会创建一个**新的提交**，内容是"把上一个提交反向改回去"；
- 历史不会被删除，所以对已经共享的提交也安全；
- 默认会打开编辑器写说明，想直接用默认说明可以加 `--no-edit`：

```bash
git revert --no-edit HEAD
```

`revert` 和 `reset` 的区别，用一句话记：**`revert` 是"再做一个相反的提交"；`reset` 是"把时间线本身挪回去"。**

### 4.5 reset 的三种模式（了解，慎用）

`git reset` 移动当前分支指针，并可选地处理暂存区/工作区：

| 模式 | 分支指针 | 暂存区 | 工作区 | 一句话 |
| --- | --- | --- | --- | --- |
| `--soft` | 移动 | 保留（改动变回已暂存） | 保留 | 只撤提交，东西都还在 |
| `--mixed`（默认） | 移动 | 清空（改动变回未暂存） | 保留 | 撤提交 + 取消暂存 |
| `--hard` | 移动 | 清空 | 清空 | **改动直接消失，危险** |

例子：撤销最近一次提交，但保留改动：

```bash
git reset --soft HEAD~1
```

`HEAD~1` 表示"HEAD 的父提交"，也就是上一个提交。

> **危险命令清单**（新手阶段看到就停一下）：
> - `git reset --hard`：丢弃工作区和暂存区的所有改动；
> - `git clean -fd`：删除所有未跟踪的文件/目录；
> - `git push --force`：覆盖远程历史；
> - `git branch -D`：强制删除未合并的分支。

### 4.6 reflog：Git 的后悔药

即使 `reset --hard` 把分支挪走了，只要那些提交曾经存在过，`git reflog` 通常还能找到：

```bash
git reflog
```

输出类似：

```text
5c1d2e3 HEAD@{0}: reset: moving to HEAD~1
9f8e7d6 HEAD@{1}: commit: practice: 一个会被 reset 掉的提交
5c1d2e3 HEAD@{2}: commit: docs: 在 README 添加学习记录
```

你可以把分支重新指回去：

```bash
git reset --hard 9f8e7d6
```

> reflog 只记录 **HEAD 的移动**，救不了从未提交过的内容。Git 默认会保留一段时间的记录（通常 30～90 天），但别把它当长期备份。

### 动手做 4：在练习分支上玩撤销

创建一个专门的练习分支，随便折腾：

```bash
git switch -c practice/undo

# A. 丢弃未暂存改动
printf '\n临时内容\n' >> README.md
git diff
git restore README.md

# B. 取消暂存
printf '\n暂存内容\n' >> README.md
git add README.md
git restore --staged README.md
git restore README.md

# C. 修正提交说明
printf '\n练习行\n' >> README.md
git add README.md
git commit -m "临时说明"
git commit --amend -m "practice: 修正后的提交说明"

# D. revert 一个提交
git revert --no-edit HEAD

# E. 用 reflog 找回
git reset --hard HEAD~1
git reflog
# 找到刚才的哈希，然后：
# git reset --hard <那个哈希>

git switch main
```

玩完可以删掉这个练习分支：`git branch -D practice/undo`（`-D` 是强制删除，因为这个分支没合并；练习分支可以这样）。

<details>
<summary>想验证 revert 做了什么？</summary>

```bash
git log --oneline -3
git show HEAD --stat
```

你会看到最新提交的说明是 `Revert "..."`，它的改动和上一个提交相反——历史里两个提交都在。

</details>

---

<a id="ch5"></a>
## 第 5 章 分支：平行时间线

### 5.1 为什么需要分支

想象你正在写一个功能，写了半天还没完成。这时线上有个 bug 要紧急修。如果所有工作都在 `main` 上，你只能：

- 把半成品提交上去（污染主线），或者
- 把半成品删掉（心疼）。

分支让你可以开一条"平行时间线"：`feature/xxx` 上慢慢写，`main` 保持干净可发布。写完再合并回去。

### 5.2 基本命令

```bash
git branch                    # 列出本地分支，* 表示当前分支
git branch -vv                # 列出分支 + 跟踪的远程 + 最新提交
git switch main               # 切换到 main
git switch -c feature/login   # 创建并切换到新分支
git branch -d feature/login   # 删除已合并的分支
git branch -D feature/login   # 强制删除（未合并也行，慎用）
git branch -m old new         # 重命名分支
```

`git switch` 是 Git 2.23 引入的新命令，比老的 `git checkout` 语义更清楚。看到教程里 `git checkout -b`，等价于 `git switch -c`。

### 5.3 切换分支时，文件会变

这是新手第一个"啊哈"时刻。跟着做：

```bash
git switch main
git status                       # 确认干净（除了那份未跟踪的手册）
git switch -c docs/learning-log
printf '\n### 分支练习\n\n- 这一行是在分支上添加的。\n' >> README.md
git add README.md
git commit -m "docs: 在分支上添加学习日志小节"
tail -5 README.md                # 能看到新加的内容
git switch main
tail -5 README.md                # 新加的内容不见了！
```

**为什么不见了？** 因为切换分支时，Git 用 `main` 指向的那个提交**重新布置工作区**。文件不是"被删了"，而是"你正在看另一个时间线的版本"。切回 `docs/learning-log`，内容又会出现：

```bash
git switch docs/learning-log
tail -5 README.md
git switch main
```

> **注意**：切换分支时如果有未提交的改动，Git 可能会带着它们一起走（如果目标分支没有冲突），也可能拒绝切换。养成习惯：**切换前先 `git status`**。

### 5.4 合并：fast-forward

现在把分支合回 main：

```bash
git switch main
git merge docs/learning-log
```

因为 `main` 在分支创建后没有新提交，Git 只需要**把 main 指针快进到分支的最新提交**，这叫 **fast-forward（快进合并）**。不会产生新的合并提交。

看历史：

```bash
git log --oneline --graph --decorate --all
```

```text
* 7a8b9c0 (HEAD -> main, docs/learning-log) docs: 在分支上添加学习日志小节
* 5c1d2e3 docs: 在 README 添加学习记录
* a3164a7 (origin/main, origin/HEAD) Initial commit
```

两个分支标签指向同一个提交。合并完成，可以删掉分支：

```bash
git branch -d docs/learning-log
```

> 想强制保留一个合并提交（团队有时要求），用 `git merge --no-ff docs/learning-log`。它会产生一个显式的 merge commit，历史图更清楚地显示"这里合过一个分支"。

### 5.5 分叉与冲突

当两条分支**都**在共同祖先之后提交了，合并就不能快进了，Git 会做"三方合并"：

```
        main:  A ── B ── E
                      \
        feature:       C ── D
```

如果两边改的是不同文件/不同行，Git 能自动合成一个合并提交；如果**改了同一行**，Git 不敢替你决定，就会报冲突（conflict）。

### 动手做 5（进阶，可选）：亲手制造并解决一次冲突

先确保在 `main` 上并且干净：

```bash
git switch main
git status
```

制造分叉：

```bash
# 1. 从 main 开一个分支
git switch -c conflict-demo

# 2. 在分支上改 README 第一行
sed -i '1s/.*/# learn-git（分支版）/' README.md
git add README.md
git commit -m "docs: 分支上修改标题"

# 3. 回到 main，也改第一行（不同的内容）
git switch main
sed -i '1s/.*/# learn-git（主分支版）/' README.md
git add README.md
git commit -m "docs: 主分支上修改标题"

# 4. 合并，预期冲突
git merge conflict-demo
```

你会看到：

```text
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

`git status` 会说 `You have unmerged paths`，并列出 `both modified: README.md`。

打开 README.md，会看到冲突标记：

```text
<<<<<<< HEAD
# learn-git（主分支版）
=======
# learn-git（分支版）
>>>>>>> conflict-demo
```

含义：

- `<<<<<<< HEAD` 到 `=======` 之间：**当前分支（main）**的版本；
- `=======` 到 `>>>>>>> conflict-demo` 之间：**要合进来的分支**的版本。

解决方式：和对方（或未来的自己）商量后，保留想要的内容，**删掉三行标记**。比如保留主分支版：

```text
# learn-git（主分支版）
```

然后：

```bash
git add README.md
git commit -m "merge: 解决 README 标题冲突"
```

或者用 `git merge --continue` 继续完成合并。之后：

```bash
git log --oneline --graph --decorate --all
git branch -d conflict-demo
```

历史图里会出现一个有两个父提交的合并提交：

```text
*   9f0e1d2 (HEAD -> main) merge: 解决 README 标题冲突
|\
| * 7c6b5a4 (conflict-demo) docs: 分支上修改标题
* | 5c1d2e3 docs: 主分支上修改标题
|/
* a3164a7 Initial commit
```

> **后悔药**：如果合并到一半不想继续，`git merge --abort` 会回到合并前的状态（未提交的冲突解决会丢失）。

> **冲突不可怕**：它只是说明"两边都动了同一处，Git 需要你做决定"。解决冲突的实质是**编辑文件**，不是敲什么神秘命令。

### 5.6 分支命名习惯

推荐 `<类型>/<简短描述>`：

- `feature/login`、`feature/readme-guide`
- `fix/typo-in-readme`
- `docs/license-section`
- `chore/update-gitignore`

好处：看名字就知道在做什么；避免 `test`、`tmp`、`我的分支` 这类模糊名字。

---

<a id="ch6"></a>
## 第 6 章 `.gitignore`：告诉 Git 哪些东西不用管

### 6.1 为什么需要它

仓库里有些东西**不该进版本历史**：

- 密钥和配置：`.env`、`secrets.toml`；
- 依赖和构建产物：`node_modules/`、`dist/`、`__pycache__/`；
- 日志和临时文件：`*.log`、`*.tmp`；
- 操作系统垃圾：`Thumbs.db`、`.DS_Store`；
- 编辑器个人配置：`.vscode/`（团队可选）。

它们要么太大、要么每次运行都变、要么包含敏感信息。`.gitignore` 就是一份"门卫名单"。

### 6.2 你现在的 `.gitignore`

你的文件来自 GitHub 的 Python 模板，4921 字节。快速看看它管了什么：

```bash
grep -nE '^(\.env|__pycache__/|\*\.log|venv/)' .gitignore
```

大致会看到第 2 行的 `__pycache__/`、第 60 行的 `*.log`、第 153 行的 `.env`。

> 模板只是普通文本，不是魔法。你可以删、可以加、可以合并多个模板。

### 6.3 规则语法

| 写法 | 含义 |
| --- | --- |
| `# 注释` | 注释行 |
| `*.log` | 任意目录下所有 `.log` 文件 |
| `build/` | 任意层级下名为 `build` 的目录 |
| `/build/` | **仅仓库根目录**下的 `build` 目录 |
| `?.tmp` | 单个字符 + `.tmp` |
| `[0-9].txt` | 一个数字 + `.txt` |
| `**/temp` | 任意深度下的 `temp` |
| `doc/**/*.pdf` | `doc` 下任意子目录的 PDF |
| `!important.log` | 取消忽略（前面的规则先忽略、后面再排除） |
| `\!name` | 文件名真的以 `!` 开头时转义 |

几条容易踩的规则：

- `.gitignore` 只影响**未跟踪**的文件。已经提交过的文件，即使后来写进 `.gitignore`，依然会被跟踪；
- 空目录不会被 Git 跟踪；一个目录里只有被忽略的文件时，它也不会出现在 `git status` 里；
- 取反 `!` 不能把"父目录已被忽略"的文件救回来。比如 `build/` 整个被忽略，`!build/keep.txt` 无效，需要先 `!build/` 再忽略具体文件；
- 规则按顺序生效，后面的可以覆盖前面的。

### 动手做 6：验证模板规则

```bash
printf 'API_KEY=demo\n' > .env
printf 'debug line\n' > app.log
mkdir -p __pycache__
printf 'x\n' > __pycache__/demo.cpython-312.pyc

git status                       # 看不到这三个文件
git status --ignored --short     # 能看到 !! 开头的被忽略项
git check-ignore -v .env app.log __pycache__/demo.cpython-312.pyc
```

`git check-ignore -v` 是排查忽略问题的神器，输出类似：

```text
.gitignore:153:.env	.env
.gitignore:60:*.log	app.log
.gitignore:2:__pycache__/	__pycache__/demo.cpython-312.pyc
```

格式是：`规则文件:行号:规则	实际路径`。**"为什么这个文件被忽略了？"——就查它。**

### 6.4 添加自己的规则

```bash
printf '\n# 我的自定义规则\nscratch/\n*.tmp\n!important.tmp\n' >> .gitignore

mkdir -p scratch
printf 'x\n' > scratch/x.tmp
printf 'x\n' > other.tmp
printf 'x\n' > important.tmp

git status --short
git check-ignore -v scratch/x.tmp other.tmp important.tmp
```

预期：`scratch/x.tmp` 和 `other.tmp` 被忽略；`important.tmp` 被 `!important.tmp` 救回来，显示为 `??`。

练习完可以删掉它：`rm important.tmp`（被忽略的 `scratch/`、`other.tmp` 不用管，`git status` 本来就不会显示它们）。

提交这条规则：

```bash
git add .gitignore
git commit -m "chore: 添加自定义忽略规则"
```

### 6.5 陷阱：已跟踪的文件不会被忽略

这是新手最常见的坑。演示一下：

```bash
printf 'log\n' > tracked.log
git add -f tracked.log                    # -f 强制添加被忽略的文件（仅演示）
git commit -m "practice: 强制加入一个日志文件"

printf '\ntracked.log\n' >> .gitignore
printf 'more\n' >> tracked.log
git status                                 # 仍然显示 modified: tracked.log
```

原因：**忽略规则只对未跟踪文件生效**。要停止跟踪但保留本地文件：

```bash
git add .gitignore
git rm --cached tracked.log
git commit -m "chore: 忽略并停止跟踪 tracked.log（本地文件保留）"
```

`--cached` 的意思是"只从 Git 索引里移除，不删磁盘上的文件"。之后 `tracked.log` 就在忽略名单里了。

> **安全提醒**：如果曾经把 `.env` 这类密钥提交过，加进 `.gitignore` 并不会把它从历史里抹掉。最稳妥的做法是**立刻更换密钥**；要清理历史则需要 `git filter-repo` 之类的工具（进阶内容，先了解即可）。

### 6.6 三种忽略位置

| 位置 | 作用范围 | 是否提交 |
| --- | --- | --- |
| `.gitignore`（仓库根目录） | 整个仓库，团队共享 | 要提交 |
| `.git/info/exclude` | 只对你本地这个仓库生效 | 不提交 |
| 全局 `core.excludesFile` | 你所有仓库 | 不提交 |

全局忽略适合放操作系统/编辑器垃圾：

```bash
git config --global core.excludesFile "$HOME/.gitignore_global"
printf 'Thumbs.db\n.DS_Store\n' > "$HOME/.gitignore_global"
```

### 6.7 获取现成模板

GitHub 维护了一个模板库：<https://github.com/github/gitignore>。你的 Python 模板就来自那里。需要用别的时候可以下载（可选，网络不通就跳过）：

```bash
curl -L https://raw.githubusercontent.com/github/gitignore/main/Node.gitignore -o .gitignore.node
```

然后按需合并，而不是盲目覆盖你现有的文件。

> 另外还有 `.gitattributes`，用来控制换行符、语言统计等；现在不用学，知道"忽略归 .gitignore，属性归 .gitattributes"就够了。

---

<a id="ch7"></a>
## 第 7 章 LICENSE：开源许可证

### 7.1 你的 LICENSE 是什么

```bash
cat LICENSE
```

你的文件是 **MIT License**，版权行是：

```text
Copyright (c) 2026 kelebuleovo
```

MIT 是使用最广的宽松许可证之一。用大白话拆成三块：

1. **授权**：任何人拿到这份软件，都可以免费使用、复制、修改、合并、发布、分发、再许可、甚至出售。
2. **条件**：必须在所有副本或"重要部分"中保留版权声明和这份许可声明。也就是说，别人可以拿去用，但不能假装是自己写的。
3. **免责**：软件按"原样"提供，作者不为使用后果负责。

> 这不是法律意见。真要用于商业项目，重要条款请找专业人士确认。

### 7.2 许可证选型速览

| 许可证 | 特点 | 适合 |
| --- | --- | --- |
| **MIT** | 最宽松，保留声明即可 | 大多数个人/开源小项目 |
| **BSD-3-Clause** | 类似 MIT，额外禁止用作者名背书 | 学术、基础设施 |
| **Apache-2.0** | MIT 基础上增加明确的专利授权和商标条款 | 商业友好、专利敏感的项目 |
| **GPL-3.0** | 传染性 copyleft：分发修改版必须同样开源 | 希望衍生作品保持开源 |
| **AGPL-3.0** | GPL + 网络服务也触发开源 | SaaS 项目 |
| **Unlicense / CC0** | 放弃版权，进入公有领域 | 希望完全无约束 |

**如果没有 LICENSE 文件**，默认是"保留所有权利"——别人不能合法地复制、修改、分发。很多人以为"公开在 GitHub 上就是开源"，这是误解。

GitHub 会读取 LICENSE 文件内容自动识别许可证，并在仓库页面显示。

### 7.3 从零添加许可证

在 GitHub 网页上创建仓库时可以直接选模板；命令行下也可以直接写文件。MIT 的最小结构：

```text
MIT License

Copyright (c) 2026 你的名字

Permission is hereby granted, free of charge, to any person obtaining a copy
...（完整条款）
```

推荐从 <https://choosealicense.com> 复制全文，避免手打出错。

<details>
<summary>想练习"从零创建 LICENSE"？点开</summary>

在你的沙盒仓库（第 2 章末尾创建的那个 `~/Desktop/git-sandbox`）里：

```bash
mkdir -p ~/Desktop/git-sandbox
cd ~/Desktop/git-sandbox
git init -b main
cat > LICENSE <<'EOF'
MIT License

Copyright (c) 2026 你的名字

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF
git add LICENSE
git commit -m "docs: 添加 MIT 许可证"
```

`<<'EOF'` 表示"把两个 EOF 之间的所有行原样写进 LICENSE"，不会做变量替换。记得把"你的名字"改成你希望显示的名字（可以用笔名）。

</details>

### 7.4 SPDX 标识（可选）

在源代码文件头部可以写一行机器可读的标识：

```python
# SPDX-License-Identifier: MIT
```

方便工具识别，但它**不能替代**仓库根目录的 LICENSE 文件。

### 动手做 7：在 README 中注明许可证

```bash
printf '\n## 许可证\n\n本项目采用 [MIT License](LICENSE)，可自由使用、修改和分发，但需保留版权声明。\n' >> README.md
git add README.md
git commit -m "docs: README 增加许可证说明"
```

再确认一下：

```bash
git log --oneline -- LICENSE
git show a3164a7:LICENSE | head -3
```

> **关于更换许可证**：已经发布的版本不能被"收回"——别人按旧许可证获得的授权依然有效。换许可证只对之后的版本生效；如果仓库有多个贡献者，通常还需要征得他们同意。所以**尽量一开始就选对**。

---

<a id="ch8"></a>
## 第 8 章 常见报错翻译器

Git 的报错看起来吓人，其实大部分都在说三件事：**你在哪、你想干什么、下一步敲什么**。注意看 `hint:` 开头的行，它经常直接给出命令。

| 报错/提示 | 白话翻译 | 怎么办 |
| --- | --- | --- |
| `fatal: not a git repository` | 当前目录不是 Git 仓库 | `cd` 到仓库目录；或确认是否忘了 `git init` |
| `Author identity unknown` / `Please tell me who you are` | 没配置名字和邮箱 | `git config --global user.name "..."`、`user.email "..."` |
| `nothing to commit, working tree clean` | 没有可提交的改动 | 正常，不是错误 |
| `Changes not staged for commit` | 改了但没 `add` | `git add <文件>` 或 `git add .` |
| `Untracked files` | 新文件没被跟踪 | 想提交就 `git add`；不想提交就写进 `.gitignore` |
| `fatal: pathspec 'x' did not match any files` | 文件名/路径写错了 | `ls` 确认；注意大小写和相对路径 |
| `error: failed to push some refs`（non-fast-forward） | 远程有你本地没有的提交 | 先 `git pull`，解决冲突后再 `git push` |
| `CONFLICT (content): Merge conflict in ...` | 两边改了同一处 | 打开文件解决冲突 → `git add` → `git commit` |
| `warning: LF will be replaced by CRLF` | Windows 换行符转换提示（你的 `core.autocrlf=true`） | 通常忽略即可，不是错误 |
| `You are in 'detached HEAD' state` | HEAD 没指向分支，直接指向提交 | 想保留工作：`git switch -c 新分支`；不想保留：`git switch main` |
| `error: Your local changes ... would be overwritten by merge` | 未提交的改动会被覆盖 | 先 commit，或 `git stash`，或 `git restore` 放弃 |
| `fatal: A branch named 'x' already exists` | 同名分支已存在 | `git switch x` 或换个名字 |
| `fatal: refusing to merge unrelated histories` | 两个仓库的历史没有共同祖先 | 一般说明操作有问题；确认无误才用 `--allow-unrelated-histories` |
| `fatal: remote origin already exists` | 已经有叫 origin 的远程 | `git remote set-url origin <新地址>` 或 `git remote rename` |
| `Permission denied (publickey)` | SSH 公钥不被认可 | `ssh -T git@github.com` 测试；检查密钥是否加到 GitHub |
| `warning: adding embedded git repository` | 目录里套了一个带 `.git` 的仓库 | 通常删掉内层 `.git` 或改用 submodule |
| `fatal: bad revision 'HEAD~2'` | 没有那么多历史提交 | `git log` 看看实际有几个提交 |

> **读报错的顺序**：先看最后一行（结论），再看 `hint:`（建议），最后看中间（细节）。很多新手只看到一大段红字就慌了，其实最后一句已经写清楚了。

---

<a id="ch9"></a>
## 第 9 章 自测清单与综合练习

### 9.1 能力清单

学完前八章，你应该能独立完成下面这些。做不到的，回到对应章节补一补。

- [ ] 解释工作区、暂存区、本地仓库、远程仓库的关系
- [ ] 解释 `HEAD`、分支指针、提交哈希分别是什么
- [ ] 用 `git status` 判断一个文件处于 untracked / modified / staged 哪种状态
- [ ] 用 `git diff`、`git diff --staged` 看懂一次改动
- [ ] 完成 `add → commit → log` 的完整循环
- [ ] 写一条信息量足够的提交说明
- [ ] 用 `git restore` 丢弃工作区改动
- [ ] 用 `git restore --staged` 取消暂存
- [ ] 用 `git commit --amend` 修正提交说明/补文件
- [ ] 用 `git revert` 撤销一个已提交的改动
- [ ] 说出 `reset --soft/--mixed/--hard` 的区别
- [ ] 创建、切换、合并、删除分支
- [ ] 独立解决一次简单的合并冲突
- [ ] 读懂 `.gitignore` 规则，并用 `git check-ignore -v` 排查
- [ ] 解释 MIT 许可证允许什么、要求什么
- [ ] 看到常见报错能自己判断下一步

### 9.2 综合练习：做一个"自我介绍"功能分支

目标：把前面学的串起来，完成一次完整的小型工作流。

1. 回到干净的 `main`：

   ```bash
   git switch main
   git status
   ```

2. 创建功能分支：

   ```bash
   git switch -c feature/self-intro
   ```

3. 新建 `intro.md`：

   ```bash
   printf '# 自我介绍\n\n- 我是谁：\n- 我在学：Git 命令行\n- 下一个目标：把本地仓库安全地同步到 GitHub\n' > intro.md
   ```

   > `printf` 里的 `\n` 是换行。`>` 表示写入（覆盖），`>>` 表示追加。这是 Git Bash 的写法，和 Git 本身无关。

4. 修改它，提交两次，每次只做一件小事。第二条提交说明要能说清"为什么"：

   ```bash
   printf '\n## 我为什么学 Git\n\n为了给学习过程留下可回溯的记录。\n' >> intro.md
   git add intro.md
   git commit -m "docs: 添加自我介绍的骨架"
   # 再改一处，再提交
   ```

5. 回到 `main`，确认 `intro.md` 不在（因为还没合并）。

6. 合并，并强制生成一个合并提交（练习 `--no-ff`）：

   ```bash
   git switch main
   git merge --no-ff feature/self-intro -m "merge: 加入自我介绍"
   ```

7. 查看历史图，确认能看到分叉和合并：

   ```bash
   git log --oneline --graph --decorate --all
   ```

8. 删除功能分支：

   ```bash
   git branch -d feature/self-intro
   ```

9. （可选）把这份手册也提交了：

   ```bash
   git add git-learning-guide.md
   git commit -m "docs: 添加 Git 学习手册"
   ```

验收标准：

- `git log --graph` 里有一个合并提交，两个父提交；
- `git branch` 里没有 `feature/self-intro`；
- `git status` 干净（或只剩你有意留下的未跟踪文件）；
- 你能不看手册，向别人解释每一步在做什么。

### 9.3 给自己出题

- 如果把第 4 步的两次提交合成一次，该用 `--amend` 还是 `revert`？为什么？
- 如果合并时冲突了，你打算按什么顺序处理？
- 如果发现提交里混进了 `practice-note.txt`，但还没 push，怎么把它从这次提交里拿掉？（提示：`git restore --staged` + `--amend`）
- 如果发现 `.env` 曾经被提交过，光加 `.gitignore` 够不够？

---

<a id="appendix-a"></a>
## 附录 A 远程仓库：下一步

你说 SSH 已经配好了，这很好。等你把本地操作练熟，再开始推送。

### A.1 确认远程

```bash
git remote -v
```

```text
origin	git@github.com:kelebuleovo/learn-git (fetch)
origin	git@github.com:kelebuleovo/learn-git (push)
```

`origin` 只是别名，真正的地址是后面那串。测试 SSH：

```bash
ssh -T git@github.com
```

看到 `Hi kelebuleovo! You've successfully authenticated...` 就说明通了（不需要 shell 权限，提示 `does not provide shell access` 是正常的）。

### A.2 推送

你的 `main` 已经跟踪 `origin/main`（克隆时就设置好了），所以直接：

```bash
git push
```

第一次在别的新仓库推送时，才需要显式建立跟踪：

```bash
git push -u origin main
```

`-u` 是 `--set-upstream`，让本地 `main` 记住"以后默认推到 `origin/main`"。

推送前用 `git status` 看：

```text
Your branch is ahead of 'origin/main' by 3 commits.
```

推送后这个提示会消失。

### A.3 fetch 和 pull 的区别

| 命令 | 做什么 | 类比 |
| --- | --- | --- |
| `git fetch` | 把远程最新提交下载到本地，但**不改你的工作区/分支** | 把快递取到门口，先不拆 |
| `git pull` | `fetch` + `merge`（或 `rebase`），直接更新当前分支 | 取回来并拆开装好 |

你的全局配置是 `pull.rebase=false`，所以 `git pull` 会做合并。团队里也常见 `git pull --rebase`，让历史更线性。新手先用默认的 merge 没问题。

推荐的安全习惯：先 `git fetch`，再 `git log --oneline HEAD..origin/main` 看看远程多了什么，最后决定 `merge` 还是 `rebase`。

### A.4 推送被拒绝（non-fast-forward）

场景：远程有别人（或你在网页上）提交的新内容，你本地没有。`git push` 报：

```text
! [rejected] main -> main (non-fast-forward)
error: failed to push some refs
```

处理：

```bash
git pull            # 把远程的拿下来并合并
# 如果有冲突，解决冲突、add、commit
git push
```

**不要**为了省事用 `git push --force`——那会覆盖远程历史，可能让别人丢失提交。只有在你确认这个分支只有你一个人用、且确实需要改写历史时，才考虑 `--force-with-lease`（它比 `--force` 安全一点）。

### A.5 网络出问题怎么办

推送/拉取失败不一定是 Git 的问题，可能是网络、DNS 或代理。先看报错最后一行的关键词：

- `Could not resolve host` → DNS/网络问题；
- `Connection timed out` → 网络不通；
- `Permission denied (publickey)` → 密钥问题。

本地学习不受影响：`add`、`commit`、`branch`、`merge` 全部离线可用。等网络恢复再 `push` 即可，提交不会丢。

---

<a id="appendix-b"></a>
## 附录 B PowerShell 差异与 Windows 小贴士

这份手册默认用 Git Bash。如果你用 PowerShell，Git 命令完全一样，区别只在**文件操作**：

| 操作 | Git Bash | PowerShell |
| --- | --- | --- |
| 查看文件 | `cat README.md` | `Get-Content README.md` |
| 创建空文件 | `touch note.txt` | `New-Item note.txt` |
| 写入一行 | `echo "内容" > note.txt` | `Set-Content note.txt "内容"` |
| 追加一行 | `echo "内容" >> note.txt` | `Add-Content note.txt "内容"` |
| 创建目录 | `mkdir -p a/b` | `New-Item -ItemType Directory a/b` |
| 列目录 | `ls -la` | `Get-ChildItem -Force` |
| 当前路径 | `pwd` | `Get-Location` |

路径写法：

- Git Bash：`~/Desktop/learn-git`、`/c/Users/Administrator/...`
- PowerShell：`$HOME\Desktop\learn-git`、`C:\Users\Administrator\...`
- Git 本身两种都认，但命令里的 `/` 在 PowerShell 参数中有时需要引号。

**换行符**：你的配置 `core.autocrlf=true` 表示：仓库里统一存 LF，检出到 Windows 工作区时转成 CRLF。看到 `LF will be replaced by CRLF` 警告是正常的。团队协作时更现代的做法是提交 `.gitattributes` 并设 `* text=auto`，这个以后再说。

**编辑器**：如果不想每次都 `-m`，可以设置默认编辑器（前提是 `code` 命令在 PATH 里）：

```bash
code --version                 # 先确认可用
git config --global core.editor "code --wait"
```

`--wait` 让 Git 等你关闭编辑器再继续。新手阶段建议还是先用 `-m`。

---

<a id="appendix-c"></a>
## 附录 C 命令速查表

### 配置

```bash
git config --global user.name "名字"
git config --global user.email "邮箱"
git config --global init.defaultBranch main
git config --list --show-origin
```

### 查看状态与历史

```bash
git status
git status --short
git status --ignored
git diff
git diff --staged
git diff HEAD
git log --oneline --graph --decorate --all
git log --stat
git log -p -2
git log -- README.md
git show HEAD
git show HEAD:README.md
git reflog
```

### 暂存与提交

```bash
git add <文件>
git add .
git add -p                 # 交互式挑块暂存（进阶）
git commit -m "说明"
git commit --amend -m "新说明"
git commit --amend --no-edit
```

### 撤销

```bash
git restore <文件>                 # 丢弃工作区改动
git restore --staged <文件>        # 取消暂存
git revert --no-edit HEAD          # 反向提交
git reset --soft HEAD~1            # 撤销提交，保留暂存
git reset HEAD~1                   # 撤销提交，取消暂存，保留工作区
git reset --hard HEAD~1            # 危险：全部丢弃
```

### 分支与合并

```bash
git branch
git branch -vv
git switch <分支>
git switch -c <新分支>
git merge <分支>
git merge --no-ff <分支>
git merge --abort
git branch -d <分支>
git branch -D <分支>
git branch -m <旧名> <新名>
```

### 忽略与文件跟踪

```bash
git check-ignore -v <路径>
git ls-files                       # 列出所有被跟踪的文件
git rm --cached <文件>             # 停止跟踪，保留本地文件
```

### 远程

```bash
git remote -v
git fetch
git pull
git push
git push -u origin main
```

### 求助

```bash
git help <命令>
git <命令> --help
git help -a
```

---

<a id="appendix-d"></a>
## 附录 D 术语中英对照

| 英文 | 中文 | 一句话解释 |
| --- | --- | --- |
| repository | 仓库 | 一个被 Git 管理的项目目录（含 `.git`） |
| working tree | 工作区 | 你眼前能编辑的文件 |
| index / stage | 暂存区 / 索引 | 打包台，决定下次提交包含什么 |
| commit | 提交 | 一次快照，有哈希、作者、说明、父提交 |
| branch | 分支 | 指向某个提交的可移动指针 |
| HEAD | 头指针 | 指向你当前所在的分支（或提交） |
| detached HEAD | 游离头指针 | HEAD 直接指向提交，不在分支上 |
| merge | 合并 | 把另一条分支的提交并入当前分支 |
| fast-forward | 快进 | 无分叉时直接移动分支指针 |
| conflict | 冲突 | 两边改了同一处，需要人工决定 |
| remote | 远程仓库 | 托管在别处的仓库副本 |
| origin | 远程别名 | 默认给远程地址起的名字 |
| upstream | 上游 | 本地分支跟踪的远程分支 |
| hash | 哈希 | 提交/对象的 40 位十六进制指纹 |
| reflog | 引用日志 | HEAD 的移动记录，找回提交的后悔药 |
| tracked / untracked | 已跟踪 / 未跟踪 | 是否已被 Git 纳入版本管理 |
| ignore | 忽略 | 让 Git 不跟踪匹配的文件 |
| stash | 储藏 | 临时收起未提交改动的功能（进阶） |
| rebase | 变基 | 把提交"搬"到另一个基点上（进阶） |

---

<a id="appendix-e"></a>
## 附录 E 学习节奏建议

如果每天只学 20～30 分钟，可以这样安排：

| 时间 | 内容 | 产出 |
| --- | --- | --- |
| 第 1 天 | 第 0～2 章 | 完成第一次命令行提交 |
| 第 2 天 | 第 3～4 章 | 能读懂 status/diff，能撤销常见错误 |
| 第 3 天 | 第 5 章 | 完成一次快进合并；尝试冲突练习 |
| 第 4 天 | 第 6～7 章 | 自定义 .gitignore，README 加许可证说明 |
| 第 5 天 | 第 8～9 章 + 综合练习 | 独立完成功能分支工作流 |
| 第 6 天（可选） | 附录 A | 第一次 push，在 GitHub 上看到自己的提交 |

**沙盒建议**：想练破坏性命令（`reset --hard`、`clean`、`rebase`）时，去一个专门的沙盒仓库：

```bash
mkdir -p ~/Desktop/git-sandbox
cd ~/Desktop/git-sandbox
git init -b main
```

在那里怎么折腾都不心疼。练熟了再回真实项目。

**老师的三个习惯**：

1. **敲任何命令前先 `git status`**——知道自己站在哪，比记住命令更重要。
2. **提交前 `git diff --staged`**——确认这次到底打包了什么。
3. **不确定就查**——`git help <命令>`、`git status` 的提示、`git check-ignore -v`，Git 自己会告诉你很多答案。

---

> 这份手册本身现在还是一个**未跟踪文件**。等你学完第 2 章，可以用刚学的命令决定是否提交它——这算是你给自己上的第一课。
>
> 祝你学得踏实，出错也不慌。