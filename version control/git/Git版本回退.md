# Git版本回退

工作区（写代码的地方，git还未记录改变化）-> add -> 暂存区 -> commit-> 本地仓库 -> commit -> 远程仓库

## 1. 已经提交commit，没有push

### 1. 1） git reset --soft撤销commit

> 已经提交commit，没有push，如果想要撤销本次提交commit，修改一下在提交，就可以使用 git reset --soft 版本号即可。这样的操作本不会造成自己写的代码丢失，只需要在一次commit即可。



### 1.2）git reset --mixed 撤销commit和add两个操作

> 这样操作会使得commit和add操作都会被撤销，回到当前版本的上一个版本，同时自己写的代码也会被撤销而消失不见的。



## 2. 已经提交commit，并且push

### 2.1）git reset --hard 撤销并舍弃版本号之后的提交记录（慎用）

> 如果有提交：A -> B ->  C -> D
>
> 如果使用：git reset --hard B
>
> 那么提交记录C D都会被舍弃而丢失的



### 2.2）git revert 撤销。但保留了提交记录

> 如果有提交：A -> B ->  C -> D 如果想要回退B的提交，同时还要保留C D提交，就用到git revert
>
> 撤销后记录会变成
>
> A -> B ->  C -> D ->B'