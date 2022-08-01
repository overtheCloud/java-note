### CentOS 安装 Ngnix

#### 官方  http://nginx.org/en/linux_packages.html 

1. 

```shell
sudo yum install yum-utils
```

2. 

```shell
vim /etc/yum.repos.d/nginx.repo
# 粘贴以下内容到上面的文件中
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

3. 

```shell
sudo yum-config-manager --enable nginx-mainline
```

4. 

```shell
sudo yum install nginx
```

