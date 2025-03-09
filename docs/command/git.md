# git使用

1. 本地分支同步到指定的远程分支

   - `git reset --hard origin/devV2.1`

2. 工作区暂存和恢复

   - `git stash `

   - `git stash pop`

3. git cherry-pick解决冲突

   - `git cherry-pick [commitID]`
   
4. 将某个commit应用到当前分支

   - 未冲突文件已合并；冲突文件搜HEAD
   - `git add [file]`
   - `git cherry-pick --continue`

5. 去掉某个历史commit

   - `git revert [commitID]`

6. 查看commit的更改

   - `git show [commitID]`

   - `git show HEAD`

7. 将当前分支回退到某个commit

   - `git reset --soft/hard [commidID]`

8. 查看远程仓库有哪些分支

   - `git branch -r`

9. 查看暂存区的修改

   - `git diff --staged`

10. git mm将本地分支提交到指定远程分支

    - `git mm upload --dest=devV2.1 openbmc/phosphor-pid-control`

11. 查看git配置(~/.gitconfig)

    - `git config --list`

12. git send-email发送补丁

    - 生成最近三次提交的补丁: `git format-patch -3`

    - 指定补丁生成的目录: `git format-patch -o /tmp/ HEAD^`

    - 发送补丁，抄送；默认会抄送给补丁提交人: `git send-email *.patch --to=xxx --cc=xxx`

    - 禁止任意自动抄送；且多个补丁分开发[PATCH 1/3]: `git send-email *.patch --to=xxx@xxx.com --suppress-cc=all`

    - 配置
    
      ```shell
      git config --global sendemail.smtpEncryption tls
      git config --global sendemail.smtpServer mail.xxx.xxx.com
      git config --global sendemail.smtpUser xxx
      git config --global sendemail.smtpPass xxx
      git config --global sendemail.smtpServerPort 587
      ```
    
13. 应用补丁

    - `git am *.path`

14. 查看每个提交记录的改动

    - `git log -p`

    - `git log --oneline`

15. 查看某个文件是在哪个提交

    - `git log init/main.c`

    - `git log --oneline init/main.c`

16. 配置git交互式使用的编辑器

    - `git config --global core.editor vim`

17. 将远程仓库回退到本地仓库的commit

    - `git push origin --force`

18. misc

    - `git add -p `

    - `git commit -s -v`

    - `git checkout -p [file]`

    - `git log --pretty=oneline --abbrev-commit`

    - `git format-patch -1 HEAD`