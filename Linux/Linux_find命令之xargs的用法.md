#linux find命令之xargs的用法
## 一 find命令应用举例
```
/> ls -l #列出当前目录下所包含的测试文件
-rw-r--r--. 1 root root 48217 Nov 12 00:57 install.log
-rw-r--r--. 1 root root  37 Nov12 00:56 testfile.dat
-rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
-rw-r--r--. 1 root root 183 Nov 1108:02 users
-rw-r--r--. 1 root root 279 Nov 1108:45 users2
```
### 1. 按文件名查找：  
-name: 查找时文件名大小写敏感。  
-iname: 查找时文件名大小写不敏感。    

*该命令为find命令中最为常用的命令，即从当前目录中查找扩展名为.log的文件。需要说明的是，缺省情况下，find会从指定的目录搜索，并递归的搜索其子目录。*  

```
/> find . -name "*.log"
 ./install.log
/> find . -iname U* #如果执行find . -name U*将不会找到匹配的文件
users users2
```
### 2. 按文件时间属性查找：
-atime -n[+n]: 找出文件访问时间在n日之内[之外]的文件。  
-ctime -n[+n]: 找出文件更改时间在n日之内[之外]的文件。  
-mtime -n[+n]: 找出修改数据时间在n日之内[之外]的文件。  
-amin -n[+n]: 找出文件访问时间在n分钟之内[之外]的文件。  
-cmin -n[+n]: 找出文件更改时间在n分钟之内[之外]的文件。  
-mmin -n[+n]: 找出修改数据时间在n分钟之内[之外]的文件。

```
/> find -ctime -2 #找出距此时2天之内创建的文件
.
./users2
./install.log
./testfile.dat
./users
./test.tar.bz2
/> find -ctime +2 #找出距此时2天之前创建的文件
没有找到#因为当前目录下所有文件都是2天之内创建的
/> touch install.log #手工更新install.log的最后访问时间，以便下面的find命令可以找出该文件
/> find . -cmin -3   #找出修改状态时间在3分钟之内的文件。
install.log
```
### 3. 基于找到的文件执行指定的操作：
**-exec:** 对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {} \;，注意{}和\；之间的空格，同时两个{}之间没有空格  
**-ok:** 其主要功能和语法格式与-exec完全相同，唯一的差别是在于该选项更加安全，因为它会在每次执行shell命令之前均予以提示，只有在回答为y的时候，其后的shell命令才会被继续执行。需要说明的是，该选项不适用于自动化脚本，因为该提供可能会挂起整个自动化流程。    

*找出距此时2天之内创建的文件，同时基于find的结果，应用-exec之后的命令，即ls -l，从而可以直接显示出find找到文件的明显列表。*

```
/> find . -ctime -2 -exec ls -l {} \;
-rw-r--r--. 1 root root  279 Nov 11 08:45 ./users2
-rw-r--r--. 1 root root  48217 Nov 12 00:57./install.log
-rw-r--r--. 1 root root  37 Nov 12 00:56 ./testfile.dat
-rw-r--r--. 1 root root  183 Nov 11 08:02 ./users
-rw-r--r--. 1 root root  10530 Nov 11 23:08./test.tar.bz2
```
 
 *找到文件名为\*.log,同时文件数据修改时间距此时为1天之内的文件。如果找到就删除他们。有的时候，这样的写法由于是在找到之后立刻删除，因此存在一定误删除的危险。*  
 
```
/> ls
install.log  testfile.dat  test.tar.bz2 users  users2
/> find . -name "*.log" -mtime -1 -exec rm -f {} \; 
/> ls
testfile.dat  test.tar.bz2  users  users2
```
在控制台下，为了使上面的命令更加安全，我们可以使用-ok替换-exec，见如下示例：

```
/>  find . -name "*.dat" -mtime -1 -ok rm -f {} \;
< rm ... ./testfile.dat > ? y#对于该提示，如果回答y，找到的*.dat文件将被删除，这一点从下面的ls命令的结果可以看出。
/> ls
test.tar.bz2  users  users2
```
### 4. 按文件所属的owner和group查找：
-user: 查找owner属于-user选项后面指定用户的文件。  
! -user:   查找owner不属于-user选项后面指定用户的文件。  
-group:   查找group属于-group选项后面指定组的文件。  
! -group: 查找group不属于-group选项后面指定组的文件。 

```
/> ls -l   #下面三个文件的owner均为root
-rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
-rw-r--r--. 1 root root 183 Nov 11 08:02 users
-rw-r--r--. 1 root root 279 Nov 11 08:45 users2
/> chown stephen users   #将users文件的owner从root改为stephen。
/> ls -l
-rw-r--r--. 1 root   root 10530 Nov 11 23:08 test.tar.bz2
-rw-r--r--. 1 stephen root183 Nov 11 08:02 users
-rw-r--r--. 1 root   root 279 Nov 11 08:45 users2
/> find . -user root #搜索owner是root的文件
.
./users2
./test.tar.bz2
/> find . ! -user root   #搜索owner不是root的文件，注意!和-user之间要有空格。
./users
/> ls -l   #下面三个文件的所属组均为root
-rw-r--r--. 1 root  root 10530 Nov 11 23:08 test.tar.bz2
-rw-r--r--. 1 stephen root183 Nov 11 08:02 users
-rw-r--r--. 1 root  root279 Nov 11 08:45 users2
/> chgrp stephen users#将users文件的所属组从root改为stephen
/> ls -l
-rw-r--r--. 1root   root10530 Nov 11 23:08test.tar.bz2
-rw-r--r--. 1 stephen stephen  183 Nov 11 08:02 users
-rw-r--r--. 1 rootroot   279 Nov 1108:45 users2
/> find . -group root   #搜索所属组是root的文件
.
./users2
./test.tar.bz2
/> find . ! -group root #搜索所属组不是root的文件，注意!和-user之间要有空格。
./users
``` 
### 5. 按指定目录深度查找：
-maxdepth: 后面的参数表示距当前目录指定的深度，其中1表示当前目录，2表示一级子目录，以此类推。在指定该选项后，find只是在找到指定深度后就不在递归其子目录了。下例中的深度为1，表示只是在当前子目录中搜索。如果没有设置该选项，find将递归当前目录下的所有子目录。

```
/> mkdir subdir  #创建一个子目录，并在该子目录内创建一个文件
/> cd subdir
/> touch testfile
/> cd ..
#maxdepth后面的参数表示距当前目录指定的深度，其中1表示当前目录，2表示一级子目录，以此类推。在指定该选项后，find只是在找到指定深度后就不在递归其子目录了。下例中的深度为1，表示只是在当前子目录中搜索。如果没有设置该选项，find将递归当前目录下的所有子目录。
/> find . -maxdepth 1 -name "*"
.
./users2
./subdir
./users
./test.tar.bz2
#搜索深度为子一级子目录，这里可以看出子目录下刚刚创建的testfile已经被找到
/> find . -maxdepth 2 -name "*"  
.
./users2
./subdir
./subdir/testfile
./users
./test.tar.bz2
```
### 6. 排除指定子目录查找：
-path pathname -prune:   避开指定子目录pathname查找。  
-path expression -prune: 避开表达中指定的一组pathname查找。  
需要说明的是，如果同时使用-depth选项，那么-prune将被find命令忽略。  
*为后面的示例创建需要避开的和不需要避开的子目录，并在这些子目录内均创建符合查找规则的文件。*

```
/> mkdir DontSearchPath  
/> cd DontSearchPath
/> touch datafile1
/> cd ..
/> mkdir DoSearchPath
/> cd DoSearchPath
/> touch datafile2
/> cd ..
/> touch datafile3
#当前目录下，避开DontSearchPath子目录，搜索所有文件名为datafile*的文件。
/> find . -path "./DontSearchPath" -prune-o -name "datafile*" -print
./DoSearchPath/datafile2
./datafile3
#当前目录下，同时避开DontSearchPath和DoSearchPath两个子目录，搜索所有文件名为datafile*的文件。
/> find . \( -path "./DontSearchPath" -o-path "./DoSearchPath" \) -prune -o -name "datafile*"-print
./datafile3
```
### 7. 按文件权限属性查找：
-perm mode:  文件权限正好符合mode(mode为文件权限的八进制表示)。  
-perm +mode: 文件权限部分符合mode。如命令参数为644(-rw-r--r--)，那么只要文件权限属性中有任何权限和644重叠，这样的文件均可以被选出。  
-perm -mode: 文件权限完全符合mode。如命令参数为644(-rw-r--r--)，当644中指定的权限已经被当前文件完全拥有，同时该文件还拥有额外的权限属性，这样的文件可被选出。

```
/> ls -l
-rw-r--r--. 1root   root   0 Nov 12 10:02datafile3
-rw-r--r--. 1root   root10530 Nov 11 23:08 test.tar.bz2
-rw-r--r--. 1 stephenstephen183 Nov 11 08:02 users
-rw-r--r--. 1root   root279 Nov 11 08:45 users2
/> find . -perm 644 #查找所有文件权限正好为644(-rw-r--r--)的文件。
./users2
./datafile3
./users
./test.tar.bz2
/> find . -perm 444 #当前目录下没有文件的权限属于等于444(均为644)。
/> find . -perm -444#644所包含的权限完全覆盖444所表示的权限。
.
./users2
./datafile3
./users
./test.tar.bz2
/> find . -perm +111#查找所有可执行的文件，该命令没有找到任何文件。
/> chmod u+x users#改变users文件的权限，添加owner的可执行权限，以便于下面的命令可以将其找出。
/> find . -perm +111 
.
./users
```

### 8. 按文件类型查找：
-type：后面指定文件的类型。  
b -块设备文件。  
d -目录。  
c -字符设备文件。  
p -管道文件。  
l  -符号链接文件。  
f  -普通文件。  

```
/> mkdir subdir 
/> find . -type d  #在当前目录下，找出文件类型为目录的文件。
./subdir
 /> find . ! -type d#在当前目录下，找出文件类型不为目录的文件。
./users2
./datafile3
./users
./test.tar.bz2
/> find . -type f  #在当前目录下，找出文件类型为文件的文件
./users2
./datafile3
./users
./test.tar.bz2
```
### 9. 按文件大小查找：
-size [+/-]100[c/k/M/G]: 表示文件的长度为等于[大于/小于]100块[字节/k/M/G]的文件。  
-empty: 查找空文件。  

```
/> find . -size +4k -exec ls -l {} \;  #查找文件大小大于4k的文件，同时打印出找到文件的明细
-rw-r--r--. 1 root root 10530 Nov 11 23:08 ./test.tar.bz2
/> find . -size -4k -exec ls -l {} \;  #查找文件大小小于4k的文件。
-rw-r--r--. 1rootroot 279 Nov 11 08:45 ./users2
-rw-r--r--. 1rootroot0 Nov 12 10:02./datafile3
-rwxr--r--. 1 stephen stephen 183 Nov 11 08:02 ./users
/> find . -size 183c -exec ls -l {} \; #查找文件大小等于183字节的文件。
-rwxr--r--. 1 stephen stephen 183 Nov 11 08:02 ./users
/> find . -empty  -type f -exec ls -l {} \;
-rw-r--r--. 1 root root 0 Nov 12 10:02 ./datafile3
```

### 10. 按更改时间比指定文件新或比文件旧的方式查找：
-newer file1 ! file2： 查找文件的更改日期比file1新，但是比file2老的文件。

```
/> ls -lrt   #以时间顺序(从早到晚)列出当前目录下所有文件的明细列表，以供后面的例子参考。
-rwxr--r--. 1 stephen stephen   183 Nov 11 08:02 users1
-rw-r--r--. 1root  root279 Nov 11 08:45 users2
-rw-r--r--. 1root   root 10530 Nov 11 23:08 test.tar.bz2
-rw-r--r--. 1root  root0 Nov 12 10:02 datafile3
/> find . -newer users1 #查找文件更改日期比users1新的文件，从上面结果可以看出，其余文件均符合要求。
./users2
./datafile3
./test.tar.bz2
/> find . ! -newer users2   #查找文件更改日期不比users1新的文件。
./users2
./users
#查找文件更改日期比users2新，但是不比test.tar.bz2新的文件。
/> find . -newer users2 ! -newer test.tar.bz2
./test.tar.bz2
```

## 二 xargs命令:
该命令的主要功能是从输入中构建和执行shell命令。  
在使用find命令的-exec选项处理匹配到的文件时， find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。  
find命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部，不像-exec选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。  
在有些系统中，使用-exec选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高；  
而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。  

```
/> ls -l
-rw-r--r--. 1 root root   0 Nov 12 10:02 datafile3
-rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
-rwxr--r--. 1 root root183 Nov 1108:02 users
-rw-r--r--. 1 root root279 Nov 11 08:45users2
#查找当前目录下的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件。
/> find . -type f -print | xargs file 
./users2:ASCIItext
./datafile3:  empty
   ./users:  ASCII text
./test.tar.bz2: bzip2 compressed data, block size = 900k
#回收当前目录下所有普通文件的执行权限。
/> find . -type f -print | xargs chmod a-x
/> ls -l
-rw-r--r--. 1 root root 0 Nov 1210:02 datafile3
-rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
-rw-r--r--. 1 root root   183 Nov 11 08:02 users
-rw-r--r--. 1 root root   279 Nov 11 08:45 users2
#在当面目录下查找所有普通文件，并用grep命令在搜索到的文件中查找hostname这个词
/> find . -type f -print | xargs grep "hostname" 
#在整个系统中查找内存信息转储文件(core dump)，然后把结果保存到/tmp/core.log文件中。
/> find / -name "core" -print | xargs echo "">/tmp/core.log
/> pgrep mysql |xargs kill -9#直接杀掉mysql的进程
[1]+ Killed mysql
```
### 1、多行变成单行
```
-bash-3.2# cattest.txt
a b c d e f
g o p q
-bash-3.2# cattest.txt |xargs
a b c d e f g op q
```
### 2、单行变成多行
```
-bash-3.2# cattest.txt
a b c d e f g op q
-bash-3.2# cattest.txt |xargs -n 2
a b
c d
e f
g o
p q
```
### 3、删除某个重复的字符来做定界符
```
-bash-3.2# cattest.txt
aaaagttttgyyyygcccc
-bash-3.2# cattest.txt |xargs -d g
aaaa tttt yyyycccc
```
### 4、删除某个重复的字符来做定界符后，变成多行
```
-bash-3.2# cattest.txt |xargs -d g -n 2
aaaa tttt
yyyy cccc
```
### 5、用find找出文件以txt后缀，并使用xargs将这些文件删除
```
-bash-3.2# find/root/ -name "*.txt" -print   #查找
/root/2.txt
/root/1.txt
/root/3.txt
/root/4.txt
-bash-3.2# find/root/ -name "*.txt" -print0 |xargs -0 rm -rf   #查找并删除
-bash-3.2# find/root/ -name "*.txt" -print #再次查找没有
```
### 6、查找普通文件中包括thxy这个单词的
```
-bash-3.2# find/root/ -type f -print |xargs grep "thxy"
/root/1.doc:thxy
```
### 7、查找权限为644的文件，并使用xargs给所有加上x权限
```
-bash-3.2# find/root/ -perm 644 -print
/root/1.c
/root/5.c
/root/2.doc
/root/3.doc
/root/1.doc
/root/2.c
/root/4.doc
/root/4.c
/root/3.c
-bash-3.2# find/root/ -perm 644 -print|xargs chmod a+x
-bash-3.2# find/root/ -perm 755 -print
/root/1.c
/root/5.c
/root/2.doc
/root/3.doc
/root/1.doc
/root/2.c
/root/4.doc
/root/4.c
/root/3.c
```
find 命令可以防止list too long 的错误，而且可以递归查找子目录，对于查找文件非常方便。  
不过有时候功能强大也会比较麻烦，例如你不想让find 命令查找子目录，只查找当前目录

跳过'src/emacs'和它下边的所有文件,列出其它发现的文件,执行下边的命令:  
`find . -path './src/emacs' -prune -o -print`  
只查找当前目录下，不搜索任何当前目录下的所有子目录  
`find . -maxdepth 1 -name "*sql"`  
使用find命令的-exec参数，即可巧妙删除。  
步骤如下：  
1、根据文件的时间，创建人，大小等特征，用find命令找到文件  
`find . -maxdepth 1 -type f -size +72019c -size -72021c `  
解释：  
-maxdepth 1 搜索深度为1  
-type f 搜索普通文件   
-size +72019c 文件大于 72019byte  
-size  -72021c 文件小于 72021byte  
2、添加 -exec 选项
`find . -maxdepth 1 -type f -size +72019c -size -72021c  -exec rm {} \;`

 