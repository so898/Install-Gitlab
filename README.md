Gitlab是一个用Ruby on Rails开发的开源项目管理程序，可以通过WEB界面进行访问公开的或者私人项目。它和Github有类似的功能，能够浏览源代码，管理缺陷和注释。

下面介绍如何在 Debian/Ubuntu 和 Centos 下搭建配置 GitLab。

##安装依赖

####Debian/Ubuntu下：

>sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server redis-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate

安装python(注意需要2.5以上版本)：

>sudo apt-get install -y python python-docutils

安装git（注意需要1.7.10以上版本）：

>sudo apt-get install -y git-core

Centos下官方仓库的软件比较老旧，推荐先添加epel源，然后再安装依赖：

>sudo yum install git patch gcc-c++ readline-devel zlib-devel libffi-devel openssl-devel make autoconf automake libtool bison libxml2-devel libxslt-devel libyaml-devel git python python-docutils

##安装 Ruby 2.0

需要安装Ruby2.0，软件仓库中的Ruby 1.8不支持：

>mkdir /tmp/ruby && cd /tmp/ruby

>curl --progress ftp://ftp.ruby-lang.org/pub/ruby/2.0/>ruby-2.0.0-p353.tar.gz | tar xz

>cd ruby-2.0.0-p353

>./configure --disable-install-rdoc

>make

>sudo make install

需要注意的是，Ruby 2+的版本在Ubuntu上会出现无法编译的情况，编译时会出现编译不成功的提示：

>make[2]: Entering directory '/tmp/ruby-build.20140317220101.16966/ruby-2.1.0/ext/ripper'

>compiling ripper.c

>installing default cparse libraries

>linking shared-object racc/cparse.so

>readline.c: In function ‘Init_readline’:

>readline.c:1977:26: error: ‘Function’ undeclared (first use in this function)
>     rl_pre_input_hook = (Function *)readline_pre_input_hook;
                          ^

>readline.c:1977:26: note: each undeclared identifier is reported only once for each function it appears in
readline.c:1977:36: error: expected expression before ‘)’ token

>   rl_pre_input_hook = (Function *)readline_pre_input_hook;
                                    ^
>readline.c: At top level:

>readline.c:634:1: warning: ‘readline_pre_input_hook’ >defined but not used [-Wunused-function]
 readline_pre_input_hook(void)
 ^

>Makefile:228: recipe for target 'readline.o' failed

>make[2]: *** [readline.o] Error 1

>make[2]: Leaving directory '/tmp/ruby-build.20140317220101.16966/ruby-2.1.0/ext/readline'
exts.mk:198: recipe for target 'ext/readline/all' failed

>make[1]: *** [ext/readline/all] Error 2

这时，我们需要手动修正readline.c文件：

1. 打开 /ext/readline/readline.c
2. 找到 (Function *) 所在位置
3. 将 Function 修改为 rl_hook_func_t

之后就可以继续进行编译安装了

安装Bundler Gem：

>sudo gem install bundler --no-ri --no-rdoc

##配置gitlab-shell

创建git用户：

>sudo adduser --system --create-home --comment 'GitLab' git  

##配置gitlab-shell

>su - git -c "git clone https://github.com/gitlabhq/gitlab-shell.git"  

>su - git -c "cd gitlab-shell && git checkout v1.3.0"  

>su - git -c "cp gitlab-shell/config.yml.example gitlab-shell/config.yml"  

>sed -i "s/localhost/gitlab.51yip.com/g" /home/git/gitlab-shell/config.yml  

>su - git -c "gitlab-shell/bin/install"  

>chmod 600 /home/git/.ssh/authorized_keys  

>chmod 700 /home/git/.ssh

##数据库

GitLab支持 MySQL 和 PostgreSQL 数据库。下面以 MySQL为例，介绍安装方法：

Debian/Ubuntu下使用如下命令安装：

>sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev

Centos下使用如下命令：

>sudo yum install mysql-server 

>sudo chkconfig mysqld on

配置MySQL：

>sudo echo "CREATE DATABASE IF NOT EXISTS gitlabhq_production DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_unicode_ci';" | mysql -u root 

>sudo echo "UPDATE mysql.user SET Password=PASSWORD('123456') WHERE User='root'; FLUSH PRIVILEGES;" | mysql -u root 

注意，用你的密码替换**123456**。

安装配置 gitlab

>su - git -c "git clone https://github.com/gitlabhq/gitlabhq.git gitlab"  

>su - git -c "cd gitlab;git checkout 5-1-stable"  

>su git -c "cp config/gitlab.yml.example config/gitlab.yml"  

>su git -c "mkdir /home/git/gitlab-satellites"  

>su git -c "mkdir public/uploads"  

>su git -c "mkdir -p tmp/sockets/"  

>su git -c "mkdir -p tmp/pids/"  

>sed -i "s/ host: localhost/ host: gitlab.yiqiding.com/g" config/gitlab.yml  

>sed -i "s/from: gitlab@localhost/from: gitlab@gitlab.yiqiding.com/g" config/gitlab.yml  

>su git -c "cp config/puma.rb.example config/puma.rb"  

>su git -c 'git config --global user.name "GitLab"'  

>su git -c 'git config --global user.email "gitlab@gitlab.yiqiding.com"'

注意将**gitlab.yiqiding.com**替换为你自己的内容。

配置数据库连接：

>sudo su git -c "cp config/database.yml.mysql config/database.yml"

>sudo sed -i "s/secure password/mysql的root密码/g" config/database.yml

安装MySQL需要的Gems

>sudo -u git -H bundle install --deployment --without development test postgres aws

初始化：

>sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production

>sudo curl --output /etc/init.d/gitlab https://raw.github.com/gitlabhq/gitlabhq/6-0-stable/lib/support/init.d/gitlab

>sudo chmod +x /etc/init.d/gitlab

>sudo update-rc.d gitlab defaults 21

查看是否配置妥当：

>sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production

重启GitLab：

>sudo service gitlab start

配置Nginx

Debian/Ubuntu下：

>sudo apt-get install -y nginx

CentOS下：

>sudo yum install nginx

下载配置文件样例：

>sudo curl --output /etc/nginx/sites-available/gitlab https://raw.github.com/gitlabhq/gitlabhq/6-0-stable/lib/support/nginx/gitlab

>sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab

修改 /etc/nginx/sites-available/gitlab，特别留意将 YOUR_SERVER_FQDN 改成自己的。

重启nginx:

>sudo service nginx restart

好了，你可以登录GitLab了，默认安装后的用户名：admin@local.host，密码5iveL!fe。

相关资料：

[Gitlab 安装方法](http://segmentfault.com/a/1190000000345686)

[Ruby 编译报错处理](https://github.com/sstephenson/ruby-build/issues/526)

[Gitlab 6+ 编译完成后binary安装](http://www.linuxidc.com/Linux/2013-08/89292.htm)
