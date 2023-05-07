# AWS linux 2023

前提：
你是个前端开发人员，你用 Node.js 和 VUE 开发了一些APP。
已经建立好了 AWS Linux2023 实例，开通了所有端口（可以后期调整）。
下载好了 PEM 私钥文件，并妥善保存。更好的办法是 GitHub Actions，自动部署。以后再讲。
AWS 是自己的改的 Linux，安全性较高，Nginx 的权限无法访问你的 dist。

## 通过 SSH 进入实例

注意：用户一般是“ec2-user”，一般不用改。

### 升级所有的包：

```
sudo yum update
```

### 安装 web 服务器

```
sudo yum install nginx
```

### 启动 ngnix，同时设定随系统启动。

```
sudo systemctl start nginx
```

### 浏览器打开IP地址（不要https）

检查一下 nginx 是否工作了。

### 安装 Node.js 和 npm

```
sudo yum install -y nodejs npm
```

### 全局安装 Vue CLI

```
sudo npm install -g vue-cli
```

### 安装 git

```
sudo yum install git
```

### 克隆你的仓库

```
git clone 仓库
```

### 给nginx这个用户权限（非常重要）

#### 把 nginx 加入 ec2-user 用户组

```
sudo usermod -a -G ec2-user nginx
```

#### 文件或目录的所有者和所属组都更改为ec2-user

这将为组成员添加目录访问权限，以确保 Nginx 用户可以访问您的项目目录（克隆出来的文件夹）。

```
sudo chown -R ec2-user:ec2-user /path/to/XXX
```

> chown命令用于更改文件或目录的所有者和/或所属组。在这个例子中，-R选项表示递归更改，即对指定路径（/path/to/XXX）下的所有文件和目录进行操作。ec2-user:ec2-user表示将文件或目录的所有者和所属组都更改为ec2-user。sudo用于执行需要管理员权限的命令。

#### 为所有者添加读写执行的权限（很重要，不要随便给root权限，网上有瞎搞的）

```
sudo chmod -R 775 /path/to/XXX
```
> chmod命令用于更改文件或目录的权限。在这个例子中，-R选项同样表示递归更改。775表示要设置的权限，它是一个八进制数，表示所有者、组和其他用户的权限。7表示读、写和执行权限（二进制表示为111），而5表示读和执行权限（二进制表示为101）。因此，775意味着所有者拥有读、写和执行权限，组拥有读、写和执行权限，其他用户拥有读和执行权限。sudo再次用于执行需要管理员权限的命令。

#### 为了保证权限的确实给了（可选）

```
chmod g+x /path/to/XXX
```

#### 进入刚才克隆好的文件夹

```
cd XXX
```

#### 进入安装依赖的包

```
npm install
```

#### 配置虚拟内存（如果你的是丐版实例，不然难以 build）

```
# 创建一个名为 swapfile 的 4G 大小的文件
sudo fallocate -l 4G /swapfile

# 设置正确的权限
sudo chmod 600 /swapfile

# 设置为 swap 文件
sudo mkswap /swapfile

# 启用 swap
sudo swapon /swapfile

# 检查 swap 是否启用
sudo swapon --show
```

#### build~~~~~

```
npm build
```

#### 配置 nginx

##### 删除缺省配置（有些时候没有，或者备份一下）

```
sudo rm /etc/nginx/conf.d/default.conf
```

##### 创建一个配置文件

```
sudo nano /etc/nginx/conf.d/beike-admin.conf
```

##### 粘贴进去，记得按下y保存

```
server {
  listen 80;
  server_name XX.XXX.XXX.XXX;

  root /home/ec2-user/XXX/dist;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }

  location /css/ {
    expires 1y;
    add_header Cache-Control "public";
  }

  location /js/ {
    expires 1y;
    add_header Cache-Control "public";
  }

  location ~* \.(jpg|jpeg|png|gif|ico|svg)$ {
    expires 1y;
    add_header Cache-Control "public";
  }

  error_page  500 502 503 504  /50x.html;
}


```

#### 这将重新加载 Nginx 以使更改生效

```
sudo systemctl reload nginx
```

# 看看你的网站能访问了吗？


