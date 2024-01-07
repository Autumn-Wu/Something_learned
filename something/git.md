# 鉴权
目前git已经废弃通过帐号密码的方式进行鉴权
## SSH
  目前主要通过SSH鉴权的方式，访问git，获取git相关权限。
配置步骤：
1. 生成密钥(入参为git邮箱，输出到指定的rsa文件中，名字自定义)
```
cd ~/.ssh
ssh-keygen -t rsa -C "$user@email.com" -f ~/.ssh/github_id_rsa
```
2. 在github中填入公钥内容：[https://github.com/settings/keys](https://github.com/settings/keys)
```
cd ~/.ssh && cat github_id_rsa.pub
```
3. 配置SSH代理：～/.ssh/config(关键在于IdentityFile的配置)
```
Host github.com
	HostName github.com 
	Port 22  
	IdentityFile ~/.ssh/github_id_rsa
```
4. 修改访问相应git仓库时的访问方式：～/.gitconfig(通过insteadOf指定访问方式)
```
[url "git@github.com:"]
  insteadOf = https://github.com/
[url "git@github.com:"]
  insteadOf = http://github.com/
```
**在经过以上步骤后，访问github对应的git仓库，会自动走ssh鉴权，并且通过配置好的ssh公钥密钥进行鉴权访问。**
## HTTPS(不建议使用)
## 小技巧
  可以做到在访问特定目录时，切换gitconfig(如切换user等)。
方法：
1. 建立特定目录(目录位置可自定义)：
```
cd ~ && mkdir TestProject
```
2. 到达用户目录：查看git配置 并新增自定义配置
```
cd ~ && cat .gitconfig
vim .gitconfig_test 
```
3. 自定义.gitconfig_test中的内容，可以为别的user等，然后绑定到gitconfig中。
在.gitconfig末尾添加
```
[includeIf "gitdir:~/TestProject/"] 
	path = .gitconfig_test
```
4. 验证是否生效(进到特定目录下clone的项目内执行)
```
git config user.name
```
# 常用操作
git 操作练习地址：[git常用操作练习](https://learngitbranching.js.org/?tdsourcetag=s_pctim_aiomsg&locale=zh_CN)
很多方法支持入参使用相对引用或commit_id。
指定参数为branch-name时，其实参数相当于指定了branch-name的head最新提交。
## 分支操作
git merge branch-name(将branch-name对应分支，合并到当前分支中)

git rebase branch-name(将branch-name分支，顺序地应用到当前分支中，并移动HEAD指针)

git branch -f branch-name 相对引用,commit_id(将branch-name的头指针，指向相对引用或指定commit_id的位置)

git branch branch-name 相对引用,commit_id(创建一个分支，指向对应相对引用的位置或commit_id的位置)

git rebase -i HEAD~n(rebase交互式操作，可以移动，删除，合并某些commit。操作范围为相对引用的commit范围)
## 切换操作
git checkout branch-name(切换到对应分支)
**此时的HEAD指向为：HEAD -> branch-name -> branch-name最新的commit**

git checkout -b branch-name(切换到对应分支，如果不存在则创建对应分支)

git checkout commit_id(将HEAD指向到指定commit)
**此时的HEAD指向为：HEAD -> commit_id**

**可借助相对引用**
git checkout HEAD^(将HEAD指向当前HEAD的parent节点，若存在多个parent节点，可以在^后添加数字，用于指定第几个parent节点，可以链接其他指令如～n)
for example:
git checkout HEAD^~2(跳到第一个parent节点，再往上两个parent节点)

git checkout HEAD~n(将HEAD指向HEAD的第n个parent)

git checkout branch-name^  = git checkout branch-name + git checkout HEAD^

git checkout branch-name~n  = git checkout branch-name + git checkout HEAD~n

## 提交操作
git add file(将对应改动文件，添加到commit缓冲区中)

git commit -m (提交，添加提交信息)

git push (--force 强制以本地仓库为准) (将当前分支情况，推送到远程分支中)

git pull (拉去远程分支中的变更情况)

git cherry-pick commit_id(将对应的提交，应用到当前分支中)

git reset 相对引用(一般用于本地仓库，强制将HEAD指向到相对引用上，并且回滚路径上到所有提交)

git revert commit(回滚某个commit，提供对应的undo commit，内容为对应提交的反操作)

git commit --amend(替换当前分支的顶部并创建一个新的提交)
git commit --amend = git reset --soft HEAD^ + something + git commit -c ORIG_HEAD

git tag tag-name <commit-id> 
(指定某个commit的提交作为里程碑tag，不指定的情况下默认为HEAD，切换到tag内不能进行修改和提交，可以理解为稳定版本或者特性)

git describe <ref>
output：<tag>_<numCommits>_g<hash>
(描述离ref提交最近的tag信息，不指定情况下ref为HEAD。tag表示的是离 ref 最近的标签， numCommits是表示这个ref与tag相差有多少个提交记录，hash表示的是你所给定的ref所表示的提交记录哈希值的前几位)