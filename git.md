## git 五连
``` shell
git clone -b development https://github.com/jtt17/opensds.git
git add /opensds
git commit -m "test"
git remote add origin https://github.com/jtt17/opensds/tree/development
git push
```

### git log 查看提交日志

### git diff 文件名  查看文件更改内容

###　git reset --hard HEAD^  回退到上一个版本  HEAD^^ 上上个版本   HEAD~100前100个版本
###  git reset --hard 0x.... 跳转到指定版本 
### git reflog 查看版本历史

### 删除远程分支 
git branch -d -r origin/branchname
git push origin --delete branchname

### 重命名分支 git branch -m oldname  newname
