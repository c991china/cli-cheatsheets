# Git 速查

## 基础
    git status
    git add -p                 # 交互式暂存
    git commit -m "msg"
    git switch -c feat/x       # 新建并切换分支
    git push -u origin feat/x

## 查看
    git log --oneline --graph
    git diff main..HEAD
    git blame <file>

## 撤销
    git restore <file>         # 丢弃工作区
    git commit --amend         # 改写最近提交
    git reset --soft HEAD~1    # 回退提交保留改动

## 远程
    git remote -v
    git fetch -p               #  prune 已删远程分支
    git push --force-with-lease
