# Git撤销
如果文件已经修改，但是还没有commit，我们可以用小乌龟的revert命令进行撤销。  
如果文件已经commit，可以使用git reset --hard HEAD~1撤销上一次提交。~<index>代表撤销前几次的提交.  
此命令回退一次commit，在回退到的commit之后的所有修改都被丢弃，所以要慎重使用。  
也可以使用git reset --soft  
HEAD~1撤销上一次提交，它和hard的区别是，同样取消一次commit，但是所有的修改都保留。此方法用来修改commit的message非常方便。  
git-revert HEAD~1      一般是按照某一次的commit完全反向的进行一次commit，他的效果和git-reset --hard  
HEAD~1 && git-commit -a -m 'revert commit <commitid> xxx....' 完全一样。  
reset命令只是相当于在本地进行一次还原操作，如果想让别的分支知道你的撤销动作，请使用revert命令，它是完全反向的进行一次commit。  
