# Git 进阶速查

```bash
# 暂存当前改动
git stash && git stash pop

# 改写最近一次提交
git commit --amend

# 交互式变基最近 3 次
git rebase -i HEAD~3

# 找回误删的提交
git reflog
git checkout <sha>

# 清理已合并到 main 的本地分支
git branch --merged main | grep -v main | xargs -r git branch -d
```
