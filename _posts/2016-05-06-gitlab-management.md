---
layout: post
date: 2016-05-06
title: GitLab管理全过程
tags: Git
---

# 目录

- [设置发送邮箱](#setup-email)
- [支持域名访问](#support-domain)
- [自动备份](#auto-backup)
- [备份恢复](#restore-backup)
- [清除缓存](#cache-clear)

## <a id="setup-email"></a>设置发送邮箱
　　GitLab在使用的过程中，在很多地方都需要使用邮箱来发送通知，比如注册通知、权限变更通知、合并通知等等，因此需要为GitLab准备一个邮箱，用于发送这些通知。

　　以下演示了如何配置QQ企业邮作为GitLab的发送邮箱。

```bash
$ sudo gedit /etc/gitlab/gitlab.rb
```

　　修改以下内容。

```bash
gitlab_rails['time_zone'] = 'UTC'
# 允许使用邮箱gitlab_rails['gitlab_email_enabled'] = true
# 使用QQ企业邮的邮箱gitlab_rails['gitlab_email_from'] = 'xxxx@xxxx.com'
# 修改这个可以改变发送邮件的发送人gitlab_rails['gitlab_email_display_name'] = 'GitLab'
# 使用QQ企业邮的邮箱gitlab_rails['gitlab_email_reply_to'] = 'xxxx@xxxx.com'
# 这个好像是设置邮件的主题，设置为2，虽然我不知道其它的主题是怎么样的。gitlab_rails['gitlab_default_theme'] = 2

################################# GitLab email server settings ################################## see https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md#smtp-settings# Use smtp instead of sendmail/postfix.gitlab_rails['smtp_enable'] = truegitlab_rails['smtp_address'] = "smtp.exmail.qq.com"gitlab_rails['smtp_port'] = 465gitlab_rails['smtp_user_name'] = "xxxx@xxxx.com"gitlab_rails['smtp_password'] = "xxxxxx"gitlab_rails['smtp_domain'] = "qq.com"gitlab_rails['smtp_authentication'] = "login"gitlab_rails['smtp_enable_starttls_auto'] = truegitlab_rails['smtp_tls'] = true
```
　　修改完配置后，需要让配置生效。

```bash
$ sudo gitlab-ctl reconfigure
```
　　这样，GitLab就可以使用这个邮箱来发送邮件了。

## <a id="support-domain"></a>支持域名访问
　　使用域名作为访问地址，除了个性化、容易记之外，还有其它优点。在更换服务器的时候，只需要在域名提供商那边修改一下域名所指向的地址就可以了，所有的仓库都不需要改动。域名不仅仅可以指向外网地址，还可以指向内网地址（比如192.168.1.2），同样也是有效的。

```bash
$ sudo gedit /etc/gitlab/gitlab.rb
```
　　修改以下内容。

```bash
## Url on which GitLab will be reachable.## For more details on configuring external_url see:## https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/configuration.md#configuring-the-external-url-for-gitlab
## 将external_url改成你的域名地址external_url 'http://xxxx.com'
```
　　修改完配置后，需要让配置生效。

```bash
sudo gitlab-ctl reconfigure
```

　　去域名提供商的解析中心，将你的域名指向你的GitLab服务器地址即可。如果你的域名在阿里云上购买的，可以这么设置。

![2016-05-06-setup-domain](/assets/blog/2016-05-06-setup-domain.png)

> 我的GitLab服务器的内网IP是192.168.9.80

　　然后就可以使用你的个性域名访问GitLab了。同时，GitLab中所有仓库的地址都指向这个域名了。

## <a id="auto-backup"></a>自动备份
　　GitLab作为源代码管理器，需要好好保护源代码的安全。因此，一个自动、定期备份GitLab是需要的。

```bash
$ sudo gedit /etc/gitlab/gitlab.rb
```
　　修改以下内容。

```bash
# 自定义管理路径
gitlab_rails['manage_backup_path'] = true
# 备份存放地址gitlab_rails['backup_path'] = "/home/user/Gitlab_Backup"
# GitLab会将数据打包成tar.gz，这个是用来设置它的访问级别，保存源代码。gitlab_rails['backup_archive_permissions'] = 0644
```
　　修改完配置后，需要让配置生效。

```bash
$ sudo gitlab-ctl reconfigure
```
　　以上设置，只是设置备份的路径和权限，并没有执行备份的动作。那么要自动备份的话，我们可以借助Linux的定期任务来完成。

```bash
# 以下命令需要在超级管理员(su)权限下执行。在Ubuntu下，可以直接在命令行下输入su并输入密码进入此模式
$ crontab -e
```
　　第一次执行的时候，会让你选择编辑器。我表示记不住vim的快捷键，所以我还是使用更简单的`nano`这个编辑器来编辑。在文件的最末端，添加以下的代码

```bash
# 每天8点执行Gitlab备份任务
0 8 * * * sudo gitlab-rake gitlab:backup:create

# 每天7点55分删除3天前的备份，如果想保存一个月的备份记录，可以改成 -mtime +30
55 7 * * * sudo find /home/yerl/Gitlab_Backup -name "*.tar" -mtime +3 -exec rm -rf {} \;
```
> GitLab的备份文件文件名的格式大概是这样的`1459920857_gitlab_backup.tar`

　　OK，这样子，GitLab每天都会去备份一次源代码，并且，会自动删除3天前的备份。
## <a id="restore-backup"></a>备份恢复
　　上面讲到了，如何定期备份。备份有几种用途，第一个，可以保障源代码的安全，第二个，当我迁移服务器的时候，我可以将备份恢复到新的服务器上，这样，新的服务器就可以正常工作了。那么，这个备份应该如何使用呢？

```bash
# 停止GitLab相关服务
$ sudo gitlab-ctl stop unicorn
$ sudo gitlab-ctl stop sidekiq

# 从备份目录中里的1459920857_gitlab_backup.tar中恢复
$ gitlab-rake gitlab:backup:restore BACKUP=1459920857

# 启动Gitlab
$ sudo gitlab-ctl start
```
> 注意：GitLab的备份和恢复都需要在同一版本内进行，不同版本间的备份文件不通用。

## <a id="cache-clear"></a>清除缓存
```bash
$ sudo gitlab-rake cache:clear
```

