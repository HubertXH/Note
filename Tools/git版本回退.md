1、git本地版本回退

git reset --hard commit_id(可用 git log -oneline 查看或git log查看)

2、git 远程版本回退

git push origin HEAD --foce #远程提交回退

或者本地回退版本提交的方式
git reset --hard HEAD~1

git push --force



3、git reverse 和git reset 的区别

1）git revert 是用一次新的commit来回滚之前的commit，git reset 是直接删除指定commit。

2）在回滚这一操作上看，效果差不多。但是在日后继续merge以前的老版本时有区别。因为git revert是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，但是git reset是之间把某些commit在某个branch上删除，因⽽和⽼的branch再次merge时，这些被回滚的commit应该还会被引⼊。

3）git reset 是把HEAD向后移动了⼀下，⽽git revert是HEAD继续前进，只是新的commit的内容和要revert的内容正好相反，能够抵消要被revert的内容。

4、git reset + commit号

git reset命令后⾯是需要加2种参数的：--hard --soft

这条命令默认情况下是”–soft”。执⾏上述命令时，这该条commit号之后（时间作为参考点）的所有commit的修改都会退回到git缓冲区中。

使⽤git status命令可以在缓冲区中看到这些修改。⽽如果加上”–hard”参数，则缓冲区中不会存储这些修改，git会直接丢弃这部分内容。

但需要注意的⼀个问题是：由于这样的重置是直接在本地的修改，⽆法提交到远程服务器，如果直接丢弃的内容已经被推到远程服务器上了，则会造成本地和服务器⽆法同步的问题。

即git reset –hard只能针对本地操作，不能针对远程服务器进⾏同样操作。如果从本地删掉的内容没有推到服务器上，则不会有副作⽤；如果被推到服务器，则下次本地和服务器进⾏同步时，这部分删掉的内容仍然会回来。

⽽上⾯注意中提到的问题则可以很好的被git revert 命令解决。

git revert + commit 号该命令撤销对某个commit的提交，这⼀撤销动作会作为⼀个新的修改存储起来，这样，当你和服务器同步时，就不会产⽣什么副作⽤。