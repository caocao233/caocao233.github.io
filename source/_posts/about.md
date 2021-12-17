---
title: 项目部署
date: 2021-12-17 17:33:55
tags:
---
## 1. ssh 链接腾讯云

---

1. 创建密钥
2. 在服务器中实例信息中绑定密钥
3. 重置密码，并重启服务器
4. 在终端中输入 ssh root@公网ip

## 2. 安装部署环境

---

**2.1 安装 Java**

Jenkins 是依赖 Java 的，所以要先安装 Java 环境。

CenterOS-8 可以直接使用 dnf 命令（dnf 是包的管理工具，类似 npm）

```powershell
dnf search java-1.8
dnf install java-1.8.0-openjdk.x86_64
```

**2.2 安装 Jenkins**

因为 Jenkins 本身是没有在 dnf 的软件仓库包中，所以要连接 Jenkins 仓库

wget 是 Linux 中下载文件的一个工具，-O 表示输出到某个文件夹并且命名为 什么文件夹

rpm 全称 The RPM Package Manage，是 Linux 下的一个软件包管理器

```powershell
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

如果没有下载指定的文件夹（/etc/yum.repos.d/）下，要将 jenkins.repo 文件移动到指定的目录下：

```powershell
mv jenkins.repo /etc/yum.repos.d/
```

安装 jenkins 需要做软件合法验证：

```powershell
cd /etc/yum.repos.d/

# 导入 GPG 密钥以确保您的软件合法
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

但是这里安装 Jenkins 仍然安装不成功，需要编辑修改 jenkins.repo

```powershell
# 进入文件
vi jenkins.repo
```

点击键盘 I，编辑文件，删除 -stable

对文件进行保存退出操作：Esc → 按住 Shift + : → 输入wq → 回车

再进行 Jenkins 安装

```powershell
dnf install jenkins
```

安装成功后，启动 Jenkins

```powershell
# 启动 Jenkins
systemctl start jenkins

# 查看 Jenkins 状态
systemctl status jenkins

# 随着操作系统的启动而启动
systemctl enable jenkins

# 输入 enable 有提示的话，根据提示可以输入命令，如：
/usr/lib/systemd/systemd-sysv-install enable jenkins
```

启动后，访问公网 ip + :8080 端口，就是 Jenkins 图形化界面。

如果访问不到 8080 端口，需要在服务器（安全组或者防火墙）中添加 8080 端口

访问公网 ip + 8080 后，出现解锁 Jenkins 界面，解锁 Jenkins ：

复制页面上的文件路径，在终端中输入：

```powershell
cat 文件路径
```

得到一串密码后，在页面中管理员密码输入得到的一串密码。

安装推荐的插件或者选择插件来安装。

**2.3 安装 Nginx**

```powershell
dnf install nginx
```

启动 Nginx

```powershell
# 启动
systemctl start nginx

# 查看状态
systemctl status nginx

# 随着服务器启动而启动
systemctl enable nginx
```

**2.4 安装 git**

```powershell
dnf install git
```

## 3. 部署操作

---

```powershell
# 创建新的文件夹
mkdir mall_cms

# 进入文件夹
cd mall_cms/

# 创建文件
touch index.html
```

**3.1 修改 Nginx 配置**

可以使用 vi /etc/nginx/nginx.conf  进行编辑

推荐：也可以在 vscode 进入到 /etc/nginx/nginx.conf 文件中进行编辑

1. 修改用户，改为：user root
2. 注释 server 中，/usr/share/nginx/html 行
3. 配置 location / {}

```powershell
# 这里配置表示：在 /root/mall_cms 目录下去找 index.html 文件
server {
	location / {
		root /root/mall_cms
		index index.html
	}
}
```

1. 修改完 Nginx 配置后，重启 Nginx，`systemctl restart nginx`

## 4. 配置 Jenkins

---

1. 新建任务
2. 源码管理
3. 构建触发器 → 定时构建

触发器规则：

定时字符串从左到右分别是：分 时 日 月 周

```powershell
# 每半小时构建一次OR每半小时检查一次远程代码分支，有更新则构建
H/30 * * * *

# 每两小时构建一次OR每两小时检查一次远程代码分支，有更新则构建
H H/2 * * *

# 每天凌晨两点定时构建
H 2 * * *

# 每月15号执行构建
H H 15 * *

# 工作日，上午9点整执行
H 9 * * 1-5

# 每周1，3，5，从8：30开始，截止19:30，每4小时30分构建一次
H/30 8-20/4 * * 1,3,5
```

## 4. 修改远程服务器文件的方式

---

1. 在 vscode 中安装插件 Remote-SSH
2. 在左侧栏目中会有一个电脑的图片
3. 点击电脑图标，并在里面进行操作

注意：有些文件在搜索栏看不到时，要进入到文件夹中，在 vscode 目录中才能查看到，可以进行编辑操作
