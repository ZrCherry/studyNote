### git命令

| 命令                           | 功能                                                     |
| ------------------------------ | -------------------------------------------------------- |
| 提交                           | git add file     git commit -m "msg"                     |
| 查看工作区状态                 | git status                                               |
| 查看修改内容                   | git diff                                                 |
| 版本切换                       | git reset --hard commit_id                               |
| 查看提交历史                   | git log                                                  |
| 查看命令历史                   | git reflog                                               |
| 丢弃工作区修改                 | git checkout -- file                                     |
| 删除文件                       | git rm file                                              |
| 关联远程库                     | git remote add origin git@server-name:path/repo-name.git |
| 推送修改                       | git push origin master                                   |
| 克隆远程仓库                   | git clone                                                |
| 创建分支                       | git branch <name>                                        |
| 查看分支                       | git branch                                               |
| 切换分支                       | git checkout/switch <name>                               |
| 合并分支                       | git merge <name>                                         |
| 删除分支                       | git branch -d <name>                                     |
| 查看分支合并图                 | git log --graph                                          |
| 保存工作现场                   | git stash                                                |
| 查看工作现场列表               | git stash list                                           |
| 恢复工作列表                   | git stash apply <listname>                               |
| 删除工作列表                   | git statsh drop <listname>                               |
| 恢复并删除                     | git stash pop                                            |
| 把master分支修改复制到当前分支 | git cherry-pick <commit id>                              |
| 查看远程库信息                 | git remote (-v)                                          |
| 推送修改                       | git push origin <branch-name>                            |
| 抓取分支                       | git pull                                                 |
| 合并分支（一条线）             | git rebase <branch-name>                                 |
| 终止git rebase                 | git rebase --abort                                       |
| 创建标签                       | git tag <name>  <commit id>                              |
| 查看所有标签                   | git tag                                                  |
| 查看标签详细信息               | git show <tagname>                                       |
| 推送全部标签                   | git push origin --tags                                   |
| 删除标签                       | git tag -d <tagname>                                     |
| 删除远程标签                   | git push origin :refs/tags/<tagname>                     |

命令也可以自己重命名，例如：

~~~java
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
~~~

