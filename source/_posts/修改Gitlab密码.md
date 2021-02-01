---
title: 修改Gitlab密码
date: 2021-01-07 17:31:38
tags: GitLab
categories: [GitLab]
cover: https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210113113011.jpeg
---
## Gitlab 密码修改

使用root用户

```bash
gitlab-rails console production
```

但是在我的CentOS上是报错的，如下错误（如果执行上述命令不出现错误，就不需要理会下面的错误）

```bash
Traceback (most recent call last):
	8: from bin/rails:4:in `<main>'
	7: from bin/rails:4:in `require'
	6: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/railties-6.0.3.3/lib/rails/commands.rb:18:in `<top (required)>'
	5: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/railties-6.0.3.3/lib/rails/command.rb:46:in `invoke'
	4: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/railties-6.0.3.3/lib/rails/command/base.rb:69:in `perform'
	3: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor.rb:387:in `dispatch'
	2: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor/invocation.rb:126:in `invoke_command'
	1: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor/command.rb:27:in `run'
/opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/railties-6.0.3.3/lib/rails/commands/console/console_command.rb:95:in `perform': wrong number of arguments (given 1, expected 0) (ArgumentError)
	9: from bin/rails:4:in `<main>'
	8: from bin/rails:4:in `require'
	7: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/railties-6.0.3.3/lib/rails/commands.rb:18:in `<top (required)>'
	6: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/railties-6.0.3.3/lib/rails/command.rb:46:in `invoke'
	5: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/railties-6.0.3.3/lib/rails/command/base.rb:69:in `perform'
	4: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor.rb:387:in `dispatch'
	3: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor/invocation.rb:126:in `invoke_command'
	2: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor/command.rb:20:in `run'
	1: from /opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor/command.rb:34:in `rescue in run'
/opt/gitlab/embedded/lib/ruby/gems/2.7.0/gems/thor-0.20.3/lib/thor/base.rb:506:in `handle_argument_error': ERROR: "rails console" was called with arguments ["production"] (Thor::InvocationError)
Usage: "rails console [options]"
You have new mail in /var/spool/mail/root
```

解决此错误

```bash
cd /var/opt/gitlab/
# 然后执行
gitlab-rails console
```

此时会返回console，通过此处就可以修改密码

```bash
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
 GitLab:       13.7.1 (c97c8073a0e) FOSS
 GitLab Shell: 13.14.0
 PostgreSQL:   12.4
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.3)
irb(main):001:0> 

# 输入，可以通过username和email去寻找用户
user=User.where(username: "root").first

# 找到后返回，若找不到，则返回 => nil
=> #<User id:1 @root>

# 修改密码，如果不是纯数字，就需要使用双引号引起来
user.password=123456

# 保存
user.save!

# 退出
quit
```
