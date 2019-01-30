---
layout:     post
title:      GitLab 简易指引（二）
subtitle:   备份与恢复
author:     WeYunx
header-style: text
catalog: true
tags:
    - GitLab

---
GitLab 的备份工作主要包含配置文件备份和应用备份。

## 配置文件备份

配置文件备份需要备份`/etc/gitlab`目录。

```bash
# 压缩文件夹
sudo sh -c 'umask 0077; tar -cf $(date "+etc-gitlab-%s.tar") -C / etc/gitlab'
```

在`crontab`中创建定时任务

```bash
sudo crontab -e -u root
```

新增一条：

```bash
# 每天1点执行
0 1 * * * umask 0077; tar cfz /secret/gitlab/backups/$(date "+etc-gitlab-\%s.tgz") -C / etc/gitlab
```

也可以将语句写成脚本，通过脚本执行，比如备份的共享目录里。

**强烈建议配置文件备份目录和应用备份目录分开！**

## 应用备份

GitLab 的备份命令如下：

```bash
# 本文的操作步骤仅适用于 Omnibus 一键安装方式安装的 GitLab，下同。
sudo gitlab-rake gitlab:backup:create

# 样例结果
Dumping database tables:
- Dumping table events... [DONE]
- Dumping table issues... [DONE]
- Dumping table keys... [DONE]
- Dumping table merge_requests... [DONE]
- Dumping table milestones... [DONE]
- Dumping table namespaces... [DONE]
- Dumping table notes... [DONE]
- Dumping table projects... [DONE]
- Dumping table protected_branches... [DONE]
- Dumping table schema_migrations... [DONE]
- Dumping table services... [DONE]
- Dumping table snippets... [DONE]
- Dumping table taggings... [DONE]
- Dumping table tags... [DONE]
- Dumping table users... [DONE]
- Dumping table users_projects... [DONE]
- Dumping table web_hooks... [DONE]
- Dumping table wikis... [DONE]
Dumping repositories:
- Dumping repository abcd... [DONE]
Creating backup archive: $TIMESTAMP_gitlab_backup.tar [DONE]
Deleting tmp directories...[DONE]
Deleting old backups... [SKIPPING]
```

执行后，备份的`tar`包放置在默认的备份目录`/var/opt/gitlab/backups` 下。

同样，我们可以编辑`/etc/gitlab/gitlab.rb`来修改默认的备份目录。

```bash
# 默认的备份路径
gitlab_rails['backup_path'] = '/mnt/backups'
```

同样这里我们强烈建议双机备份，官网提供了将备份上传到云以及上传到 mount 共享文件夹下，这里介绍一下上传到共享文件夹下的配置。 

比如我在本地 windows 环境下创建了一个共享文件夹`gitlab_backups` ，然后将文件夹挂载到服务器上：

```bash
# 在根目录下创建文件夹
mkdir gitlab_backups

# 挂载
mount -t cifs -o uid=996,gid=993,username=user,password=pass //22.189.30.101/gitlab_backups /gitlab_backups
```

其中`uid`和`gid`是服务器上`git`用户的`uid`和`gid`，如果不加上很可能会报权限异常。`user`和`pass`就是你本地的用户名密码，后面的 ip 和目录就是本地的共享目录。

挂载成功后修改配置：

```bash
 # 自动将备份文件上传
 gitlab_rails['backup_upload_connection'] = {
   :provider => 'Local',
   :local_root => '/gitlab_backups'
 }

 # 配置备份文件放至在挂载文件夹里的子目录名称，如果备份文件直接放至在挂载目录里，使用 ‘.’ 
 gitlab_rails['backup_upload_remote_directory'] = '.'
```

修改完成后执行`sudo gitlab-ctl reconfigure`使配置生效，此时再执行备份命令则会自动将备份文件复制到挂载的共享目录里。

同样，我们可以加到定时任务中：

```bash
# 每天2点执行
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create
```



## GitLab 恢复

配置文件的恢复很简单，直接将备份文件替换，然后执行`sudo gitlab-ctl reconfigure`即可。

下面说一下应用备份的恢复:

首先是确认工作：

>- You have installed the **exact same version and type (CE/EE)** of GitLab Omnibus with which the backup was created.
>
>- You have run `sudo gitlab-ctl reconfigure` at least once.
>- GitLab is running. If not, start it using `sudo gitlab-ctl start`.



开始恢复：

```bash
# 复制备份文件
sudo cp 11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar /var/opt/gitlab/backups/

# 权限变更
sudo chown git.git /var/opt/gitlab/backups/11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar

# 停止服务
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq

# 确认服务停止
sudo gitlab-ctl status

# 恢复
sudo gitlab-rake gitlab:backup:restore BACKUP=1493107454_2018_04_25_10.6.4-ce

# 重启和检查
sudo gitlab-ctl restart
sudo gitlab-rake gitlab:check SANITIZE=true
```




*（完）*



## 参考

- https://docs.gitlab.com.cn/