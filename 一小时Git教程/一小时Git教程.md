# 一小时Git教程

## 2 安装和初始化配置

官网下载安装即可，安装完成后要验证是否安装成功，可以使用`git -v`来进行查看
### Git的使用方式

- 命令行
- 图形化界面（GUI）
- IDE插件/扩展

### 初始化配置

配置用户名和邮箱

```git
git config --global user.name "liuiu"
git config --global user.name liuiu
git config --global user.email liu.moon.710@gmail.com
```

—global：全局配置，设置时对所有仓库都有效
—system：系统配置，对系统中的所有用户都有效
如果不添加该选项，默认只对本地仓库有效
配置完成后，如果不想每次都输入用户名和密码，可以进行保存

```git
git config --global credential.helper store
```

查看配置的信息

```git
git config --global --list
```

## 3 新建仓库

方式一：git init
方式二：git clone

```shell
mkdir learn-git
cd learn-git
git init
git init learn-git
ls
ls -a
```

看到.git目录说明Git仓库已经创建成功了
或者可以使用方式二

```shell
git clone https://github.com/geekhall-laoyang/remote-repo.git
```

## 4 工作区域和文件状态

Git的本地数据管理分为三个区域

- 工作区（Working Directory）：电脑中的文件夹
- 暂存区（Staging Area/Index）：临时存储区域，用于保存即将提交到Git仓库的修改内容
- 本地仓库（Local Repository）：git init创建的那个仓库，包含了完整的项目历史和元数据

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/b20de35a8a90defe3bdbecbe415fd11.png)

Git中文件的状态

- 未跟踪（Untrack）：新创建的，还没有被Git管理起来的文件
- 未修改（Unmodified）：已经被Git管理起来，但是文件的内容没有发生变化，还没有被修改过
- 已修改（Modified）：已经修改了的文件，但是还没有添加到暂存区里面
- 已暂存（Staged）：修改之后，并且已经添加到了暂存区域内的文件

具体的流程如下图所示：

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/236e4555e865a518a4d75a160cc1aa5.png)

## 5 添加和提交文件

相关命令

```shell
git init        // 创建仓库
git status  // 查看仓库的状态 查看当前仓库处在哪个分支，有哪些文件以及这些文件当前处在怎样的一个状态
git add     // 添加到暂存区
git commit  // 提交 只会提交暂存区中的文件，而不会提交工作区中的其他文件
git log     // 查看提交记录
git log --oneline   // 查看简洁的提交记录
```

下面进行演示：

```shell
echo "这是第一个文件"> file1.txt
git status
git add file1.txt
git rm --cached file1.txt   // 取消提交暂存
git commit -m "这里输入提交的相关信息"
git status
```

Git支持通配符添加多个文件

```shell
git add *.txt
// 把当前文件夹下的所有文件都添加到暂存区
git add .
git log
```

## 6 git reset回退版本

git reset的三种模式

- soft：回退到某一个版本，并且保留工作区和暂存区的所有修改内容
- hard：回退到某一个版本，并且丢弃工作区和暂存区的所有修改内容
- mixed：回退到某一个版本，并且只保留工作区的修改内容、丢弃暂存区的修改内容（默认参数）

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/a55ac49fd03472a929c479cfdcaecaa.png)

```shell
git reset --soft
git reset --hard
git reset --mixed
```

首先需要查询提交历史

```shell
git log --oneline
>id 相关信息
>id 相关信息
git reset --soft id
git log --oneline
git reset --hard HEAD^  // HEAD^表示上一个版本
git log --oneline
git reset HEAR^
git log --oneline
```

一般来说，当我们提交了多个版本，但是又觉得这些提交没有太大意义，可以合并成一个版本的时候，可以通过这两个参数来进行回退之后再重新提交，他们的主要区别是在重新提交之前，混合模式需要执行一下git add操作来将变动的内容重新添加到暂存区，而soft模式就不需要了，因为暂存区并没有被清空。
而hard参数的使用场景，则一般是真的要放弃目前本地的所有修改内容的时候，谨慎使用hard这个参数
如果误操作删除了，可以查询历史版本号再回退到这个版本

```shell
git reflog  // 查看我们操作的历史记录
git reset 版本号
```

## 7 使用git diff查看差异

```shell
git diff        // 查看文件在工作区、暂存区以及版本库之间的差异 或者 查询文件在两个特定版本之间的差异   或者文件在两个分支之间的差异
```

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/cd6c459ae851e414995181512285a2c.png)

```shell
git diff        // 默认比较 工作区 和 暂存区 之间的差异内容，会显示发生更改的文件以及更改的详细信息
git diff HEAD   // 比较 工作区 和 本地仓库 之间的差异
git diff --cached   // 比较暂存区 和 本地仓库 之间的差异
git diff id1 id1        // 比较两个版本之间的差异
git diff id HEAD        // 比较id和当前分支之间的差异
git diff HEAD～ HEAD // 比较当前版本和上一个版本之间的差异
git diff HEAD～3 HEAD file3.txt  // 查看HEAD之前的3个版本和当前版本中 file3.txt文件中差异
```

注：`HEAD`表示当前分支的最新提交、`HEAD~`和`HEAD^`表示上一个版本、`HEAD~2`表示HEAD之前的两个版本，数字以此类推。

## 8 使用git rm删除文件

```shell
// 方法一：
rm file1.txt        // 在本地工作区中删除文件
git ls-files        // 查看暂存区中的内容
git add file1.txt   // 将暂存区中的文件也删除掉
git add .
git commit -m "delete file1.txt"
// 方法二：
git rm file2.txt        // 将file2.txt文件在工作区和暂存区中都删除
git status
git commit -m "delete file2.txt"
```

总结：

```shell
rm file; git add file   // 先从工作区删除文件，然后再从暂存区删除文件
git rm <file>           // 把文件从工作区和暂存区同时删除
git rm --cached <file>  // 把文件从暂存区删除，但保留在当前工作区中
git rm -r *             // 递归删除某个目录下的所有子目录和文件
                        // 删除后不要忘记提交
```

## 9 .gitignore忽略文件

.gitignore文件作用：让我们忽略掉一些不应该被加入到版本库中的文件，这样可以让我们的仓库体积更小、更加干净。
应该忽略哪些文件呢？

- 系统或者软件自动生成的文件
- 编译产生的中间文件和结果文件
- 运行时生成日志文件、缓存文件、临时文件
- 涉及身份、密码、口令、密钥等敏感信息文件

下面是一个实际的例子

```shell
echo "some log" > access.log        // 模拟日志文件
echo "other log" > other.log        // 用于对比的日志文件
git status    // 查看文件状态
echo access.log > .gitignore    // 将access.log 添加到 .gitignore 文件中
cat .gitignore    // 查看.gitignore文件
git status    // 再次查看文件状态
git add .    // 把所有修改都添加到暂存区
git status    // 再次查看文件状态
git commit -m "ignore file sample"    // 提交修改
git ls-files    // 查看仓库中的文件
vi .gitignore    // 修改.gitignore文件
    access.log
    *.log        // 表示忽略所有的以log结尾的文件
echo hello > hello.log    // 再添加一个日志文件
git status    // 查看状态
git commit -am "test ignore log"    // 再次提交
git ls-files    // 查看版本库中的文件
```

注意：.gitignore文件生效需要有一个前提，即这个文件不能是已经被添加到版本库中的文件。如果文件已经添加到版本库中了，则需要使用git rm命令删除掉，然后再进行提交的操作
如果我我们只是想把文件从版本库里面删除，而不想删除本地文件的话，就可以在后面加`—-cached`这个参数

```shell
git rm —-cached other.log    // 将other.log文件从版本库删除，而不删除本地文件
git status    // 查询文件状态
git commit -am "delete other.log"    // 提交更改
echo " some change" >> other.log     // 更改文件内容
git status    // 查看文件状态
```

.gitignore文件还可以配置文件夹的名称

```shell
mkdir temp    // 创建temp文件夹
              // 注：如果temp文件夹是空的，则不会被纳入到版本控制中
              //     如果temp文件夹下有文件的话，就会被纳入到版本控制之中
echo "hello" > temp/hello.txt    // 添加文件
git status -s    // -s short 查看简略模式
?? temp/         // 这里的两个问号，第一列表示是暂存区的状态
                 // 第二列表示是工作区的状态
vi .gitignore    // 添加temp文件夹的名称
    access.log
    *.log
    temp/        // 文件夹以斜线结尾
git status -s    // 再次查看文件状态
M .gitignore     // M表示.gitignore文件被修改过
git commit -am "test ignore folder"    // 提交修改
git ls-files     // 查看仓库文件内容
```

.gitignore文件的匹配规则

从上到下逐行匹配，每一行表示一个忽略模式

- 空行或者以#开头的行会被Git忽略。一般空行用于可读性的分隔，#一般用作注释
- 使用标准的Blob模式匹配，例如：
    - 星号\*通配任意个字符
    - 问号?匹配单个字符
    - 中括号\[]表示匹配列表中的单个字符，比如：\[abc]表示a/b/c
- 两个星号\*\*表示匹配任意的中间目录
- 中括号可以使用短中线连接，比如：
    - \[0-9]表示任意一位数字，\[a-z]表示任意一位小写字母
- 感叹号!表示取反
    - 要忽略指令模式以外的文件或者目录可以加!

例如

```shell
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a ，即使你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
/build

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

GIT官网匹配规则：

[官网地址](https://git-scm.com/docs/gitignore)
github提供的各种常用语言的忽略文件的模板，在新建仓库的时候，我们可以直接使用，也可以根据自己的需要进行修改，[文件地址](https://github.com/github/gitignore)

## 10 注册GitHub账号

## 11 SSH配置和克隆仓库

创建仓库步骤

- Create repository
- 填写Repository name
- Description
- 选择可见性
- 是否初始化一个README文件
- 是否创建一个.gitignore文件
- 开源许可证文件
- 点击新建仓库按钮

HTTPS模式和SSH模式：

- HTTPS：
    - 在我们把本地代码push到远程仓库的时候，需要验证用户名和密码
    - 在2021年8月13日以后，HTTPS这种方式已被GitHub停止使用了
- SSH：
    - 这种方式在推送的时候不需要验证用户名和密码，但是需要在GitHub上添加SSH公钥的配置
    - 这种是比较推荐的方式，更加安全和方便
  
复制地址并克隆仓库

```shell
git clone git@github.com:geekhall-luoyang/remote-repo.git
```

报错了，提示我们没有正确的访问权限，是因为我们还没有配置SSH密钥导致的，下面看看如何配置SSH密钥

```shell
cd    // 回到用户的根目录
cd .ssh    // 进入到.ssh目录
ssh-keygen -t rsa -b 4096    // -t 指定协议为RSA -b 指定生成的大小为4096
// 需要我们输入密钥的文件名称
test
// 再次回车生成密码
// 如果直接回车，默认没有密码
// 否则输入密码，并记住，在下面的步骤中还会用到
ls -ltr    // 查看本地目录
test       // 私钥文件，谁要也不给！！！
test.pub   // 公钥文件，上传到GitHub
```

我们需要打开公钥文件，然后复制公钥文件的内容，回到GitHub页面，点击右上角的头像，找到最下面的settings，在页面左侧有`SSH and GPG keys`，点开之后点击页面右侧的`New SSH key`，然后把刚刚复制的公钥内容粘贴到下面的输入框中（即Key），Title输入任意的名字（这里输入test），然后点击`Add SSH key`添加这个密钥，就成功的把密钥添加到GitHub上了。

创建config文件

```shell
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/test
```

该文件的意思是当我们访问github.com的时候，指定使用SSH下的test这个密钥。
然后回到本地仓库再次执行克隆命令

```shell
git clone git@github.com:geekhall-luoyang/remote-repo.git
// 这里提示我们要输入密码，也就是我们刚刚创建SSH密钥的时候指定输入的密码
// 如果当时直接回车，这里也直接回车
// 否则输入密码
// 克隆成功后，进入仓库
cd remote-repo
echo hello > hello.txt    // 添加文件
git add .    // 添加到暂存区
git commit -m "first commit"    // 提交到本地仓库
git ls-files    // 查看仓库状态
```

管理本地仓库和远程仓库

- git pull
    - 把远程仓库的修改拉取到本地仓库
- git push
    - 把本地仓库的修改推送给远程仓库

```shell
git push    // 把本地仓库的修改推送给远程仓库
```

总结

- 生成SSH Key
    - ssh-keygen -t rsa -b 4096
        - 私钥文件：id-rsa
        - 公钥文件：id-rsa.pub
- 克隆仓库
    - git clone repo-address
- 推送更新内容
    - git push \<remote> \<branch>
- 拉取更新内容
    - git pull \<remote>

## 12 关联本地仓库和远程仓库

如果我们本地已经有了一个仓库，怎样才能把它放到远程仓库里面呢？

首先在github上创建一个新的仓库，名称为：first-repo，直接点击创建仓库按钮
点击url右侧的按钮来复制一下地址
现在我们的目标是将本地仓库my-repo和first-repo仓库关联起来

```shell
git remote add origin git@github.com:geekhall-laoyang/first-repo.git
// 添加一个远程仓库
// origin：远程仓库的别名，一般默认的别名都是这个，也可以自己指定一个其他的名字
git remote -v    // 查看我们的当前仓库所对应的远程仓库的别名和地址
git branch -M main    // 指定分支的名称为main
git push -u origin main    // 把本地的main分支和远程仓库的main分支关联起来
git push -u origin main:main    // 上面命令的全称
    // -u upstream 把本地仓库和别名为origin的远程仓库关联起来
    // main:main 把本地仓库的main分支推送给远程仓库的main分支
    // 如果本地分支的名称和远程仓库的名称相同的话 我们就可以省略 只写一个main就可以了
// 回车之后会提示我们输入密码
```

将远程仓库的文件拉取到本地仓库
git pull \<远程仓库名> \<远程分支名>:\<本地分支名>
这里面仓库到名称和分支的名称可以省略，如果省略到话默认就是拉取仓库别名为origin的main分支
它到作用就是把远程仓库的指定分支拉取到本地再进行合并

```shell
git pull
```

注意：在执行完一次git pull之后，git会自动为我们执行一次合并操作，如果远程仓库中的修改内容和本地仓库中的修改内容没有冲突的话，那么合并操作就会成功，否则合并操作就会由于冲突而失败，这个时候我们就需要手动来解决一下冲突

从远程仓库获取内容还可以使用fetch命令，fetch和pull的区别在于，fetch只是获取远程仓库的修改，但是并不会自动合并到本地仓库中，而是需要我们手动合并

总结

- 添加远程仓库：
    1. git remote add \<远程仓库别名> \<远程仓库地址>
    2. git push -u \<远程仓库名> \<分支名>
- 查看远程仓库：
    - git remote -v
- 拉取远程仓库内容：
    - git pull \<远程仓库名> \<远程分支名>:\<本地分支名>
    - \<远程分支名>:\<本地分支名>如果相同可以省<本地分支名>

## 13 Gitee的使用和GitLab本地化部署

目前用不到，以后需要再用

## 14 GUI工具

- GitHub Desktop
- Sourcetree
- GitKraken

## 15 在VSCode中使用git

配置VSCode的命令行启动，即code .
点击 查看菜单栏 -> 命令面板 （control shift + p）-> 输入 path -> 在PATH中安装code命令

在代码管理器中四个图标的作用

- 打开文件
    - 在vscode中打开对应的文件
- 放弃更改
    - 丢弃这个文件还没提交的更改内容
- 添加到暂存区
    - 就是git add命令
- 当前文件的状态
    - ??（Untracked）：未跟踪
    - U（Untracked）：未跟踪（这个讲解和上面的有出入？？？）
    - M（Modified）：已修改
    - A（Added）：已添加暂存
    - D（Deleted）：已删除
    - R（Renamed）：重命名
    - U（Updated）：已更新未合并

提交修改：点击提交按钮，等价于git commit -m

## 16 分支简介和基本操作

分支的使用场景：

分支非常适合团队协作和开发管理，比如多个开发人员可以在自己的分支上进行开发工作，最后再合并到主线代码库中，我们也可以在一个分支上进行新功能的开发，或者建立一个问题修复的分支来处理一些bug和缺陷，这样就可以让主线代码仓库处于一个随时可用的比较稳定的状态，而不会影响到其他功能的开发和测试，保证了项目的正常运行和高效协作。分支的优点就是能够提高团队协作的效率，减少冲突和错误的影响，让团队中的每个人都能独立地开发和测试。

下面我们来一起看一下Git中分支的一些基本操作：

```shell
mkdir branch-demo
cd branch-demo
git init    // 初始化仓库
```

仓库中的文件名和提交记录我们都以最简单的方式来命名，使用分支名加序号来命名文件，使用分支名加冒号加序号的方式来编写提交记录。

```shell
echo main1 > main1.txt
git add .
git commit -m "main:1"    // main分支的第1次提交
echo main2 > main2.txt
git commit -m "main:2"    // main分支的第2次提交
echo main3 > main3.txt
git commit -m "main:3"    // main分支的第3次提交
git branch    // 查看当前仓库的所有分支
* main    // * 后面的就是目前所处的分支
git branch dev    // 创建新的分支，名为dev
git branch    // 查看当前仓库的所有分支
git checkout dev    // 切换到dev分支
// checkout 命令还有其他的功能，为了避免歧义，改用switch命令
git switch main    // 切换到main分支
git switch dev    // 切换到dev分支
echo dev1 > dev1.txt
git add .
git commit -m "dev:1"
echo dev2 > dev2.txt
git add .
git commit -m "dev:2"
git switch main
echo main4 > main4.txt
git add .
git commit -m "main:4"
echo main5 > main5.txt
git add .
git commit -m "main:5"
```

现在假如测试完成，并没有任何问题之后，我们就需要把这个dev功能的分支合并到主线代码中，可以使用git merge命令来将不同的分支合并到当前分支中

```shell
git merge dev
// dev 是将要被合并的分支
// 当前所在的分支就是合并后的目标分支
// 也就是 如果把dev分支合并到main分支中
    // 切换到main分支
    // git merge dev
```

如果想通过命令行查看分支图，可以使用git log

```shell
git log --graph --online --decorate --all
```

如果一个分支不再需要了，可以进行删除

```shell
git branch -d dev    // 删除已经完成合并到分支
// 如果一个分支已经被合并到其他分支中了，就可以使用该命令进行删除
// 但如果没有被合并 则需将 -d 变为 -D 来强制删除
git branch
```

总结：

- 查看分支列表：
    - git branch
- 创建分支：
    - git branch branch-name
- 切换分支
    - git checkout branch-name
    - git switch branch-name    （推荐）
- 合并分支
    - git merge branch-name
- 删除分支
    - 已合并
        - git branch -d branch-name
    - 未合并
        - git branch -D branch-name

## 17 解决合并冲突

一般情况下，如果两个分支的修改内容没有重合的部分的话，那么合并分支就非常简单，Git会为我们自动完成合并，但是如果两个分支修改了同一个文件的同一行代码，Git就不知道应该保留哪个分支的修改内容了，也就产生了冲突，这个时候就需要我们手动来解决冲突。

首先看一个例子，沿用上节课的仓库，新建分支feat，feat是feature的缩写，一般用来表示开发某一个功能的分支

```shell
git branch feat
git switch feat
ls
vi main1.txt
  main1
  这是feat分支中添加的内容
git commit -a -m "feat:1"    // 加上-a参数，就可以一个命令完成 添加暂存 和 提交 两个动作
git switch main
vi main1.txt
  main1
  这是main分支中添加的内容
git commit -am "main:6"
// 这时，main分支和feat分支就产生了分歧
git merge feat    // 尝试合并分支
// 提示：自动合并失败了，需要解决冲突之后在提交
git status    // 查看冲突文件的列表
git diff    // 查看冲突的具体内容
```

这时我们需要手工编辑这个文件，留下我们想要的内容之后在提交

```shell
vi main1.txt
```

将文件内容由

```txt
main1
<<<<<<< HEAD
这是main分支中添加的内容
=======
这是feat分支中添加的内容
>>>>>>> feat
```

修改为

```txt
main1
这是main分支中添加的内容,这是合并后的内容
```

之后保存退出

```shell
git add .
git commit -m "merge conflict"
// 提交之后就自动完成了合并
// 在提交之间如果想中断这次合并的话，可以使用git merger --abort 命令来终止合并
```

总结：

- 两个分支未修改同一个文件的同一处位置：Git自动合并
- 两个分支修改了同一个文件的同一处位置：产生冲突
    - 解决方法：
        1. 手工修改冲突文件，合并冲突内容
        2. 添加暂存区 git add file
        3. 提交修改 git commit -m "message"
    - 中止合并：当不想继续执行合并操作时可以使用下面的命令来中止合并过程
        - git merge --abort

## 18 回退和rebase

除了使用merge操作来合并不同的分支，还有另外一个方法可以将不同分支的修改内容整合到一起，就是rebase，中文意思是变基

下面来看一下怎样使用rebase以及在什么情况下使用rebase

之前在执行合并操作的时候，是先使用checkout或者switch命令来切换到main分支，然后执行git merge dev命令，就可以将dev分支合并到main分支上。合并完成之后的结果就是main分支上会多出一个提交记录，然后两个分支就像两条溪流一样汇聚到了一起。

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/96cab327eaaef6f1f8858db9a26f7fb.png)

那么如果使用rebase的话，就不必在main分支上执行操作了，我们可以在任意分支上执行rebase操作，可以在dev分支上，也可以在main分支上

如果我们在dev分支上执行rebase操作的话，dev分支的两次提交记录就都会变基到main分支上，而在main分支上执行的话，main分支的两次提交记录就会变基到dev分支的末尾，执行rebase之后，最后的结果都是一条直线，但是中间的顺序会稍微有些不同。

在Git中，每个分支都有一个指针，指向当前分支的最新提交记录，而在执行rebase操作的时候，Git会先找到当前分支和目标分支的共同祖先，这里也就是main3这个提交节点，再把当前分支上从共同祖先到最新提交记录的所有提交都移动到目标分支的最新提交后面

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/a888964b1c69b2593fe400503fa0368.png)

下面实际演示一下rebase操作
首先进入到之前创建的这个branch-demo的仓库中，先将仓库恢复到执行merge之前的状态，也就是main:5这次之前的提交，现将演示合并冲突的feat分支删除掉

```shell
git branch -d feat    // 删除feat分支
```

在之前合并完成之后，我们已经把dev分支删除掉了，现在需要把它恢复回来，可以执行git checkout -b dev id，这样就可以恢复到这个分支的该时间点的状态。

```shell
git checkout -b dev 224d35
```

这个提交ID可以在GitKraken中看到，可以使用git log命令来查看，注意下面的操作都是在dev分支中完成的

```shell
git log --one line --graph --decorate --all
```

如果命令太长了，可以使用alias命令来将它定义为一个别名

```shell
alias graph="git log --one line --graph --decorate --all"
```

这样就可以直接使用graph来查看图形化的提交记录了
然后我们再切换回main分支，因为main分支也需要退回到合并之前的main:5这次提交的状态

```shell
git switch main
```

可以使用reset命令来将我们的仓库回退到某一个时间点

```shell
git reset --hard b4d139
```

这样仓库的状态就恢复到了我们执行合并操作之前的状态了
为了演示两次不同的rebase操作，我们先来将这个仓库复制一下

```shell
cd ..
cp -rf branch-demo rebase1
cp -rf branch-demo rebase2
```

下面在rebase1中执行命令

```shell
git switch dev     // 切换到dev分支
git rebase main    // 将当前的dev分支变基到目标的main分支上
```

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/4a796e22d0a5bb58ef63bf27947e37a.png)

下面再来看一下在main分支上执行rebase操作的结果，我们打开rebase2这个仓库

```shell
git switch main
git rebase dev    // 将当前的main分支变基到目标的dev分支上
```

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/29301d2b0d16ae25ece11b74add32d5.png)

那么rebase和merge有什么区别，该如何区分使用呢？

- merge
    - 优点：不会破坏原分支的提交历史，方便回溯和查看
    - 缺点：会产生额外的提交节点，分支图比较复杂
- rebase
    - 优点：不会新增额外的提交记录，形成线性历史，比较直观和干净
    - 缺点：会改变提交历史，改变了当前分支branch out的节点，要避免在共享分支使用

一般来说，如果你只是想把两个分支合并起来，而不关心提交历史的话，那么就可以使用git merge命令，如果你确定只有你自己在一个分支上开发，并且希望提交历史更加的清晰明了，那么就建议使用rebase命令

## 19 分支管理和工作流模型

所谓工作流模型就是一些比较好的规范和流程，可以让我们的工作更高效、更有条理。
一个比较常用和流行的工作流模型是：GitFlow模型

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/024f8bae5ca7783936e23cb0ed8146c.png)

它将分支分成了五种类型，每种类型都有自己的用途

- 主分支（master/main）：代表了项目的稳定版本，每个提交到主分支的代码都应该是经过测试和审核的
- 开发分支（develop）：用于日常开发。所有的功能分支、发布分支和修补分支都应该从开发分支派生出来
- 功能分支（feature）：用于开发单独的功能或者特性。每个功能分支都应该从开发分支派生，并在开发完成后合并回开发分支
- 发布分支（release）：用于准备项目发布。发布分支应该从开发分支派生，并在准备好发布版本后合并回主分支和开发分支
- 热修复分支（hotfix）：用于修复主分支上的紧急问题。热修复分支应该从主分支派生，并在修复完成后，合并回主分支和开发分支

另外一个模型是GitHub Flow

![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/44022fee646289028380f3f7f1372e7.png)

GitHub Flow模型只有一个长期存在的主分支，而且主分支上的代码是可以直接部署到生产环境中的，那一般会设置分支保护，禁止团队成员直接在主分支上进行提交，团队成员可以从主分支中分离出自己的分支进行开发和测试，然后在本地分支提交代码，等到开发完成之后，可以发起一个Pull Request（简称PR、拉请求或者合并请求），团队成员们可以对代码进行Review评审，如果没有问题，就可以将这个PR发布和合并到主分支中，整个流程就完成了

还有一些比较好的习惯

- 分支命名
    - 推荐使用带有意义的描述性名称来命名分支
        - 版本发布分支/Tag示例：v1.0.0
        - 功能分支示例：feature-login-page
        - 修复分支示例：hotfix-#issueid-desc
- 分支管理
    - 定期合并已经成功验证的分支，及时删除已经合并的分支
    - 保持合适的分支数量
    - 为分支设置合适的管理权限