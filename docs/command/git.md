# git使用

1. 本地分支同步到指定的远程分支

   - `git reset --hard origin/devV2.1`

2. 工作区暂存和恢复

   - `git stash `
   - `git stash pop`

3. 将某个commit应用到当前分支

   - `git cherry-pick [commitID]`
   
4. git cherry-pick解决冲突

   - 未冲突文件已合并；冲突文件搜HEAD
   - `git add [file]`
   - `git cherry-pick --continue`

5. 去掉某个历史commit

   - `git revert [commitID]`

6. 查看commit的更改

   - `git show [commitID]`
   - `git show HEAD`
   - `git show [commitID] kernel/fork.c`

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
    - `git log --pretty=oneline --abbrev-commit`
    
15. 查看某个文件是在哪个提交

    - `git log init/main.c`
    - `git log --oneline init/main.c`
    
16. 配置git交互式使用的编辑器

    - `git config --global core.editor vim`
    
17. 将远程仓库回退到本地仓库的commit

    - `git push origin --force`
    - `git push -f`
    
18. 将本地仓库另一个仓库的commit应用到当前仓库的分支

    - 添加远程仓库
      - `git remote add other_repo /home/gxh/linux-2.4`
    - 获取远程分支
      - `git fetch other_repo`
    - 将指定commitID应用到当前仓库的当前分支
      - `git cherry-pick [commitID]`
    
19. git仓库创建

    - 不创建 .git 目录

      - 把指定目录当作 .git
        - `git clone --bare https://github.com/openbmc/dbus-sensors.git /home/gxh/dbus-sensors.git`

      - 和指定 .git 共享objects
        - `git clone -n -s /home/gxh/dbus-sensors.git/ /home/gxh/dbus-sensors`

    - 创建 .git 目录

      - 拉取仓库
        - `git clone https://github.com/openbmc/dbus-sensors.git`
      - 和 .git 目录共享objects
        - `git clone -n -s /home/gxh/dbus-sensors/.git/ /home/gxh/dbus-sensors.git`

    - checkout到指定分支的指定commitID

      - `git checkout -B master [commitID]`

    - 指定master分支的上游分支

      - `git branch master --set-upstream-to origin/master`

    - 指定当前分支的上游分支

      - `git branch --set-upstream-to origin/master`
    
20. 查看指定分支是否有指定commit

    - `git branch --contains [commitID] --list master 2> /dev/null | wc -l`
    
21. 查看tag
    - `git tag -l`
    
22. 查看指定commitID在哪些tag中
    - `git log --oneline --reverse kernel/fork.c` -> 查看commitID
    - `git tag --contains [commitID]`
    
23. linux查看tag，一个tag本质上是一个commitID的别名
    - `git log --oneline | grep "Linux .*"`
    
24. 查看一个文件指定内容在哪个commit第一次被添加
    - `git log --reverse --oneline -S"SYSCALL_DEFINE1(unshare" -- kernel/fork.c | head -n 1`
    - `git show [commitID] kernel/fork.c` -> 查看一个commit的指定文件的改动
    
25. 查看一个commit最早出现在哪个tag
    - `git describe --tags --contains [commitID] 2>/dev/null | cut -d"~" -f1 | head -n1`
    - `git describe --tags --contains [commitID] 2>/dev/null | awk -F'~' '{print $1}' | head -n1`
    
26. 查看指定文件在哪个tag第一次被添加
    - `git log --reverse --oneline --pretty=format:%H -S"SYSCALL_DEFINE1(unshare" -- kernel/fork.c | head -n1 | xargs -I{} git describe --tags --contains {} 2>/dev/null | cut -d"~" -f1 | head -n1`
    
27. 列出修改指定文件的commit

    - `git log --oneline -- socket.c`
    - 只看最近三个commit：`git log --oneline -3 -- socket.c`
    
28. 查看指定文件内容历史改动

    - `git log --oneline -p socket.c`
    
29. 查看一个commit中指定文件的改动(oneline一行显示commit信息)

    - `git show --oneline [commitID] -- socket.c`
    
30. 只查看一个commit改动的文件列表

    - `git show --oneline --name-only [commitID]`
    
31. commit信息添加 Signed-off-by

    - `git commit -s -m "test"`
    
32. 交互式添加commit信息

    - `git commit -v`