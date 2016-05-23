# 环境

```shell

[root@iZ2851te7e5Z ~]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.1.1503 (Core)
Release:	7.1.1503
Codename:	Core

```

服务器安装了

- PHP7
- Nginx,占用80端口
- Mysql

# 安装

安装采用官网提供的安装方法.

进入页面 https://about.gitlab.com/downloads/

选择 CentOS 7

## 硬件要求

GitLab对硬件的要求不是很高,很显然,越好的硬件,越能支撑起更多的项目的和用户.

## 系统要求

### 支持的类UNIX系统

- Ubuntu
- Debian
- CentOS
- Red Hat Enterprise Linux (please use the CentOS packages and instructions)
- Scientific Linux (please use the CentOS packages and instructions)
- Oracle Linux (please use the CentOS packages and instructions)

### 不支持的类UNIX系统

- OS X
- Arch Linux
- Fedora
- Gentoo
- FreeBSD

### 不是类UNIX的系统

比如Windows,并不支持.

## 安装和配置必要的依赖关系

如果你安装postfix发送邮件，请选择“网站设置”中。而不是使用后缀也可以使用sendmail配置自定义SMTP服务器配置为SMTP服务器。

```
sudo yum install curl policycoreutils openssh-server openssh-clients
sudo systemctl enable sshd
sudo systemctl start sshd
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld
```


### postfix 服务启动失败

```
 /usr/sbin/postconf: fatal: parameter inet_interfaces: no local interface found for ::1
```

修改配置文件 `vi /etc/postfix/main.cf`

修改的部分为
```
inet_interfaces = 127.0.0.1 #只能接受内部邮件，其它邮件不接受

inet_protocols = all
```

启动服务 `sudo systemctl start postfix`,成功.


### 安装firewalld

` yum install firewalld`

`systemctl unmask firewalld`


## 添加GitLab安装包到服务器

`curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash`

### 添加国内的镜像源

执行上面的命令,会一直 time out ,所以我们要换成国内的源.
 
以下操作针对CentOS 7 ,其他的请戳 [https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)

`vim /etc/yum.repos.d/gitlab-ce.repo`

```
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```

注意,如果对应配置文件下有文件`gitlab_gitlab-ce.repo`,重命名一下,不然会默认加载这个导致上面的文件不起作用.

查看目前的yum进程,并杀死

```
 ps -a
  PID TTY          TIME CMD
18781 pts/0    00:00:00 sudo
18783 pts/0    00:00:00 bash
18796 pts/0    00:00:00 yum
18855 pts/0    00:00:00 sudo
18856 pts/0    00:00:00 yum
18871 pts/0    00:00:00 ps

kill -9 18796
kill -9 18856

```

```
sudo yum makecache
sudo yum install gitlab-ce 
```

上面执行完了,是这样的展示结果

```
sudo gitlab-ctl reconfigure

gitlab: GitLab should be reachable at http://iZ2851te7e5Z
gitlab: Otherwise configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
gitlab: And running reconfigure again.
gitlab: 
gitlab: For a comprehensive list of configuration options please see the Omnibus GitLab readme
gitlab: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md
gitlab: 
It looks like GitLab has not been configured yet; skipping the upgrade script.
  验证中      : gitlab-ce-8.7.6-ce.0.el7.x86_64                                                                                         1/1 

已安装:
  gitlab-ce.x86_64 0:8.7.6-ce.0.el7                                                                                                         

完毕！

```


## 配置和开始使用GitLab


`sudo gitlab-ctl reconfigure`

接下来会自动配置文件权限,安装数据库....

提示!安装的时间会很长!!!

<del>根据我们服务器监控记录,配置过程花了5个小时!</del>

# 修改配置文件 /etc/gitlab/gitlab.rb

目前的状态是完成了安装包的安装,但是还没有启用配置文件,所以依赖还都没有装。

所以非常不建议直接运行`sudo gitlab-ctl reconfigure`,<del>不能再踩一次坑!QAQ</del>

基本我们要调的东西都在`/etc/gitlab/gitlab.rb`里面,所以这个文件一定要仔细看好。

## 修改连接数据库为Mysql

因为我们本机已经用了LNMP做了环境,所以可以直接采用Mysql作为我们的数据库,而不用`postgresql`,减少服务器的负担。

**企业版才支持使用mysql**

QAQ

```
# Disable the built-in Postgres
postgresql['enable'] = false

# Fill in the values for database.yml
gitlab_rails['db_adapter'] = 'mysql2'
gitlab_rails['db_encoding'] = 'utf8'
gitlab_rails['db_host'] = '127.0.0.1'
gitlab_rails['db_port'] = '3306'
gitlab_rails['db_username'] = 'USERNAME'
gitlab_rails['db_password'] = 'PASSWORD'
```

## 采用本机自带的nginx

```
################
# GitLab Nginx #
################
## see: https://gitlab.com/gitlab-org/omnibus-gitlab/tree/master/doc/settings/nginx.md

nginx['enable'] = false
nginx['client_max_body_size'] = '250m'
nginx['redirect_http_to_https'] = false
#nginx['redirect_http_to_https_port'] = 80
# nginx['ssl_client_certificate'] = "/etc/gitlab/ssl/ca.crt" # Most root CA's are included by default
# nginx['ssl_certificate'] = "/etc/gitlab/ssl/#{node['fqdn']}.crt"
# nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/#{node['fqdn']}.key"
# nginx['ssl_ciphers'] = "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256"
# nginx['ssl_prefer_server_ciphers'] = "on"
# nginx['ssl_protocols'] = "TLSv1 TLSv1.1 TLSv1.2" # recommended by https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html & https://cipherli.st/
# nginx['ssl_session_cache'] = "builtin:1000  shared:SSL:10m" # recommended in http://nginx.org/en/docs/http/ngx_http_ssl_module.html
# nginx['ssl_session_timeout'] = "5m" # default according to http://nginx.org/en/docs/http/ngx_http_ssl_module.html
# nginx['ssl_dhparam'] = nil # Path to dhparams.pem, eg. /etc/gitlab/ssl/dhparams.pem
nginx['listen_addresses'] = ["0.0.0.0", "[::]"]
nginx['listen_port'] =  80    # override only if you use a reverse proxy: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#setting-the-nginx-listen-port
# nginx['listen_https'] = nil # override only if your reverse proxy internally communicates over HTTP: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#supporting-proxied-ssl
# nginx['custom_gitlab_server_config'] = "location ^~ /foo-namespace/bar-project/raw/ {\n deny all;\n}\n"
# nginx['custom_nginx_config'] = "include /etc/nginx/conf.d/example.conf;"
# nginx['proxy_read_timeout'] = 300
# nginx['proxy_connect_timeout'] = 300
# nginx['proxy_set_headers'] = {
#  "Host" => "$http_host",
#  "X-Real-IP" => "$remote_addr",
#  "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
#  "X-Forwarded-Proto" => "https",
#  "X-Forwarded-Ssl" => "on"
# }
# nginx['proxy_cache_path'] = 'proxy_cache keys_zone=gitlab:10m max_size=1g levels=1:2'
# nginx['proxy_cache'] = 'gitlab'
# nginx['http2_enabled'] = true
# nginx['real_ip_trusted_addresses'] = []
# nginx['real_ip_header'] =
# nginx['real_ip_recursive'] = nil

nginx['custom_nginx_config'] = "include /etc/nginx/conf.d/*.conf;" # If you need to add custom settings into the NGINX config, for example to include existing server blocks, you can use the following setting.
## Advanced settings
nginx['dir'] = "/usr/local/nginx"
nginx['log_directory'] = "/usr/local/nginx"
nginx['worker_processes'] = 4
nginx['worker_connections'] = 10240
nginx['log_format'] = '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"'
# nginx['sendfile'] = 'on'
# nginx['tcp_nopush'] = 'on'
# nginx['tcp_nodelay'] = 'on'
nginx['gzip'] = "on"
nginx['gzip_http_version'] = "1.0"
nginx['gzip_comp_level'] = "2"
# nginx['gzip_proxied'] = "any"
nginx['gzip_types'] = [ "text/plain", "text/css", "application/x-javascript", "text/xml", "application/xml", "application/xml+rss", "text/javascript", "application/json" ]
nginx['keepalive_timeout'] = 65
nginx['cache_max_size'] = '5000m'
```

创建vhost下的配置文件,指向GitLab文件夹

```
upstream gitlab-workhorse {
  server unix://var/opt/gitlab/gitlab-workhorse/socket fail_timeout=0;
}

server {
  listen *:80;
  server_name git.example.com;
  server_tokens off;
  root /opt/gitlab/embedded/service/gitlab-rails/public;

  client_max_body_size 250m;

  access_log  /var/log/gitlab/nginx/gitlab_access.log;
  error_log   /var/log/gitlab/nginx/gitlab_error.log;

  # Ensure Passenger uses the bundled Ruby version
  passenger_ruby /opt/gitlab/embedded/bin/ruby;

  # Correct the $PATH variable to included packaged executables
  passenger_env_var PATH "/opt/gitlab/bin:/opt/gitlab/embedded/bin:/usr/local/bin:/usr/bin:/bin";

  # Make sure Passenger runs as the correct user and group to
  # prevent permission issues
  passenger_user git;
  passenger_group git;

  # Enable Passenger & keep at least one instance running at all times
  passenger_enabled on;
  passenger_min_instances 1;

  location ~ ^/[\w\.-]+/[\w\.-]+/(info/refs|git-upload-pack|git-receive-pack)$ {
    # 'Error' 418 is a hack to re-use the @gitlab-workhorse block
    error_page 418 = @gitlab-workhorse;
    return 418;
  }

  location ~ ^/[\w\.-]+/[\w\.-]+/repository/archive {
    # 'Error' 418 is a hack to re-use the @gitlab-workhorse block
    error_page 418 = @gitlab-workhorse;
    return 418;
  }

  location ~ ^/api/v3/projects/.*/repository/archive {
    # 'Error' 418 is a hack to re-use the @gitlab-workhorse block
    error_page 418 = @gitlab-workhorse;
    return 418;
  }

  # Build artifacts should be submitted to this location
  location ~ ^/[\w\.-]+/[\w\.-]+/builds/download {
      client_max_body_size 0;
      # 'Error' 418 is a hack to re-use the @gitlab-workhorse block
      error_page 418 = @gitlab-workhorse;
      return 418;
  }

  # Build artifacts should be submitted to this location
  location ~ /ci/api/v1/builds/[0-9]+/artifacts {
      client_max_body_size 0;
      # 'Error' 418 is a hack to re-use the @gitlab-workhorse block
      error_page 418 = @gitlab-workhorse;
      return 418;
  }

  location @gitlab-workhorse {

    ## https://github.com/gitlabhq/gitlabhq/issues/694
    ## Some requests take more than 30 seconds.
    proxy_read_timeout      3600;
    proxy_connect_timeout   300;
    proxy_redirect          off;

    # Do not buffer Git HTTP responses
    proxy_buffering off;

    proxy_set_header    Host                $http_host;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header    X-Forwarded-Proto   $scheme;

    proxy_pass http://gitlab-workhorse;

    ## The following settings only work with NGINX 1.7.11 or newer
    #
    ## Pass chunked request bodies to gitlab-workhorse as-is
    # proxy_request_buffering off;
    # proxy_http_version 1.1;
  }

  ## Enable gzip compression as per rails guide:
  ## http://guides.rubyonrails.org/asset_pipeline.html#gzip-compression
  ## WARNING: If you are using relative urls remove the block below
  ## See config/application.rb under "Relative url support" for the list of
  ## other files that need to be changed for relative url support
  location ~ ^/(assets)/ {
    root /opt/gitlab/embedded/service/gitlab-rails/public;
    gzip_static on; # to serve pre-gzipped version
    expires max;
    add_header Cache-Control public;
  }

  error_page 502 /502.html;
}
```



# 使用

## 查看状态

上面的命令是通过`gitlab-ctl`安装的,那么通过`gitlab-ctl`命令一样也能做别事情~

`gitlab-ctl`

```shell

I don't know that command.
/opt/gitlab/embedded/bin/omnibus-ctl: command (subcommand)
deploy-page
  Put up the deploy page
remove-accounts
  Delete *all* users and groups used by this package
upgrade
  Run migrations after a package upgrade
General Commands:
  cleanse
    Delete *all* gitlab data, and start from scratch.
  help
    Print this help message.
  reconfigure
    Reconfigure the application.
  show-config
    Show the configuration that would be generated by reconfigure.
  uninstall
    Kill all processes and uninstall the process supervisor (data will be preserved).
Service Management Commands:
  graceful-kill
    Attempt a graceful stop, then SIGKILL the entire process group.
  hup
    Send the services a HUP.
  int
    Send the services an INT.
  kill
    Send the services a KILL.
  once
    Start the services if they are down. Do not restart them if they stop.
  restart
    Stop the services if they are running, then start them again.
  service-list
    List all the services (enabled services appear with a *.)
  start
    Start services if they are down, and restart them if they stop.
  status
    Show the status of all the services.
  stop
    Stop the services, and do not restart them.
  tail
    Watch the service logs of all enabled services.
  term
    Send the services a TERM.

```

这样就知道了我们的服务怎么使用了~

## status 查看状态

```
# gitlab-ctl status
run: gitlab-workhorse: (pid 19751) 23124s; run: log: (pid 19750) 23124s
run: logrotate: (pid 31160) 1078s; run: log: (pid 19765) 23091s
run: nginx: (pid 32621) 0s; run: log: (pid 19755) 23119s
run: postgresql: (pid 19584) 23964s; run: log: (pid 19583) 23964s
run: redis: (pid 19501) 23975s; run: log: (pid 19500) 23975s
run: sidekiq: (pid 19831) 22616s; run: log: (pid 19738) 23128s
run: unicorn: (pid 19707) 23134s; run: log: (pid 19706) 23134s
```

## tail 查看日志

这个命令查看我们的gitlab在运行过程中有没有问题.

`gitlab-ctl tail`

# 后记
<del>GitLab对服务器的要求比较高,文档上说4核8G,我的1核512M的小服务器在安装多次后卡死多次。我决定暂时先放放。。。以后再做这个。。。</del>


QAQ





# 参考资料

- https://about.gitlab.com/gitlab-com/
- http://www.chhua.com/web-note4929
- https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/
- https://about.gitlab.com/downloads/
- https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/database.md#database-settings