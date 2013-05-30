# Git一些小经验
![image](http://static.oschina.net/uploads/space/2011/0608/162554_wEoO_124879.png)
查看帮助，要装git-doc，另外推荐git的图形客户端gitg，比gitk好看多了，用apt-get install就可

HEAD是当前工作版本的指针

	--global保存的是当前用户的配置，配置文件保存在~/.gitconfig  
	--system是系统中所有用户，配置文件一般在/etc/gitconfig  
什么都不加就是当前目录下项目的配置文件，在项目的.git文件夹中  
git config --list 可以查看所有配置信息，有重名的因为有不同的配置文件，实际会采用最后一个  

### 基本设置

	git config --global user.name yisen 
	git config --global user.name yisen.n@gmail.com 
	git config --global core.editor vim 
### 给git着色
git config --global color.ui true 这样会好看一些
### 自动完成脚本
git默认要输入全命令，而且还不能像svn那样st,ci,co，有点不方便  
其实在git的源代码文件夹中，contrib/completion 目录下的 git-completion.bash脚本可以实现自动完成  
把它复制到～/.git-completion.bash，然后source之，并且把命令加到启动脚本中echo "source ～/.git-completion.bash >> ~/.bashrc"
现在我们就可以用我们习惯的<tab><tab>来自动补全命令了
### Git 命令别名
	$ git config --global alias.co checkout
	$ git config --global alias.br branch
	$ git config --global alias.logg "log --pretty=format:'%h - %an -%ad -%s'"

git log -p 查看每个版本的差异  
git log a..b 查看a版本到b版本之间的log  
git reflog 可以查看每个改动  

git reset HEAD~1 撤销最近的一次改动  
恢复数据 用reflog查看已经没了的提交的SHA值，然后直接git branch recover-branch ab1afef(SHA值前几位)

	git checkout -b newbranch = git branch newbranch + git checkout newbranch 
	git checkout -b newbranch develop = git checkout develop + git checkout -b newbranch
### 永远不要rebase那些已经推送到公共仓库的更新
如果你遵循这条金科玉律，就不会出差错。否则，人民群众会仇恨你，你的朋友和家人也会嘲笑你，唾弃你。
如果把rebase当成一种在推送之前清理提交历史的手段，而且仅仅rebase那些永远不会公开的 commit，那就不会有任何问题。如果rebase那些已经公开的 commit，而与此同时其他人已经用这些 commit 进行了后续的开发工作，那你有得麻烦了。
### 忽略添加某些文件
习惯git add .来增加所有更改，如果有不想被默认添加进仓库的，可以在项目目录下新建一个.gitignore文件，把文件名输进去，空行分隔，可以用*号。
### 局域网内共享仓库
无SSH的：
本机上用`git clone --bare xxx xxx.git` 克隆一个纯仓库  
xxx.git放到服务器上一个项目组都能访问到的共享目录下，比如NFS，假设/mnt/git/xxx.git   
对方先mount到自己的/mnt/git，然后 `git clone /mnt/git/xxx.git`  
添加远程主机: `git remote add origin file://192.168.x.x/opt/xxx.git`  
获取更新: `git fetch origin` 这时得到一个origin/master分支的指针，不能修改，   
可以合并到自己的主干 git co master, git merge origin/master，或者新建一个分支来工作，`git checkout -b new_br origin/master`  
获取更新并合并到当前分支的命令可以合并为: git pull origin master  (master:master) 
### 改变最后一次提交
修改了文件后，git status看到有更改，`git checkout --` . 可以撤销这些修改
`git commit --amend`可以重新提交一次，以便更改说明 
也可以先add或者rm一下后再使用上面的命令，可以修改上次提交。因为会更改sha值，所以不要在push之后再修改
### 取回前版本
`git reset --hard/soft/mixed xxxxx`, 取消所有修改，保留所有修改，默认,清空文件状态
`git reflog` 查看所有操作log
### 暂存工作
git stash 可以把你当前工作的杂乱无张的状态先暂存起来，然后你就可以切换到别的分支去工作  
git stash list 可以看当前的暂存列表后  
git stash apply 应用最新的一个暂存，git stash drop stash@{0}来删掉暂存  
### 格式化的困扰
你可能会遇到行尾结束符问题。这是因为Windows使用回车和换行两个字符来结束一行，而Mac和Linux只使用换行一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作。  
`git config --global core.autocrlf input` 签出时不转换，提交是把CRLF转换成LF  
`git config --global core.autocrlf true` 如果服务器是linux，工作在windows，提交时自动地把行结束符CRLF转换成LF，而在签出代码时把LF转换成CRLF。   
服务器跟工作都在windows上，设成false取消此功能  
### 查看某个文件的历史修改记录
`git log -p filename` filename替换成对应的文件即可
