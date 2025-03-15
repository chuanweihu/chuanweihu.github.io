---
layout: post
title:  使用Ansible服务实现自动化运维
categories: [linux]
comments: true
tags: [Linux]
---

> 本文主要介绍Linux系统服务器的ansible管理软件。

* TOC
{:toc}

<!--more-->

## 介绍

Ansible 通过从安装和配置了 Ansible 组件的计算机（称为 Ansible 控制节点 ）配置客户端计算机（称为 托管节点 ）来工作。

它通过正常的 SSH 通道进行通信，以从远程系统检索信息、发出命令和复制文件。 因此，Ansible 系统不需要在客户端计算机上安装任何其他软件。

这是 Ansible 简化服务器管理的一种方式。 任何暴露了 SSH 端口的服务器都可以归入 Ansible 的配置保护伞，无论它处于生命周期的哪个阶段。 这意味着您可以通过 SSH 管理的任何计算机，也可以通过 Ansible 进行管理。

Ansible 采用模块化方法，使您能够扩展主系统的功能以处理特定场景。 模块可以用任何语言编写并以标准 JSON 进行通信。

配置文件主要以 YAML 数据序列化格式编写，因为它的表达性和与流行标记语言的相似性。 Ansible 可以通过命令行工具或其配置脚本（称为 Playbooks）与主机交互。

## 先决条件

- 一个 Ansible 控制节点：Ansible 控制节点是我们将用来通过 SSH 连接和控制 Ansible 主机的机器。 您的 Ansible 控制节点可以是您的本地计算机或专用于运行 Ansible 的服务器，但本指南假定您的控制节点是 Ubuntu 18.04 系统。 确保控制节点具有：
    - 具有 sudo 权限的非 root 用户。 要进行此设置，您可以按照我们的 Ubuntu 18.04 初始服务器设置指南的 步骤 2 和 3 。 但是，请注意，如果您使用远程服务器作为 Ansible 控制节点，则应遵循本指南的 每一步 。 这样做将使用 ufw 在服务器上配置防火墙并启用对您的非 root 用户配置文件的外部访问，这两者都将有助于保持远程服务器的安全。
    - 与此用户关联的 SSH 密钥对。 要进行设置，您可以按照我们关于 如何在 Ubuntu 18.04 上设置 SSH 密钥的指南中的 Step 1 进行操作。
- 一个或多个 Ansible 主机：Ansible 主机是您的 Ansible 控制节点配置为自动化的任何机器。 本指南假定您的 Ansible 主机是远程 Ubuntu 18.04 服务器。 确保每个 Ansible 主机具有：
    - Ansible 控制节点的 SSH 公钥添加到系统用户的 authorized_keys 中。 此用户可以是 root 或具有 sudo 权限的普通用户。 要进行此设置，您可以按照 如何在 Ubuntu 18.04 上设置 SSH 密钥的 步骤 2。

## 架构

Ansible 基于 Python 语言所开发，使用 Paramiko 来实现 SSH 连接，使用 PyYAML 来解析 YAML 格式的 playbook 文件，并使用 Jinja2 作为模板引擎来生成模板文件

Ansible 由控制节点和远程受管节点组成，控制节点 是用来安装 Ansible 软件、执行维护命令的服务器（ 控制节点不支持 Windows），受管节点是运行业务服务的服务器，由控制节点通过 SSH 来进行管理，受管节点可以是 Linux、OS X、类 Unix、Windows 等主机，或者 Cisco 等网络设备或负载均衡器

### 主要组件

- Host Inventory： 定义了 Ansible 能够管理的远程主机资源清单，默认为 /etc/ansible/hosts 文件。也可以编写一个 inventory plugin 来连接到任何返回 JSON 的数据源（比如你的 CMDB）
- Modules： Ansible 执行命令的功能模块，包括内置核心模块或用户自定义模块
- Plugins： 功能模块的补充，比如上面的 inventory plugin，或者提供转换数据、记录输出、日志记录等附加功能。插件只能使用 Python 编写
- API： 提供给第三方应用调用的接口

### 特性

- 模块化： Ansible 依靠 模块（Modules） 来实现批量部署，模块就是实现了指定功能的 Python 脚本程序，通过 SFTP 或 SCP 拷贝到远程受管节点的临时目录中，执行后（返回 JSON 数据）会被删除。Ansible 内置了大量的模块，使用它们可以完成软件包安装、重启服务、拷贝配置文件等诸多操作
- 支持自定义模块： 由于模块支持可插拔，所以你也可以开发自己的模块来扩展它的功能，可以使用任何能够返回 JSON 数据的语言（比如 Python、Ruby、Shell 等）来编写专用模块
- 部署简单： 基于 Python 和 SSH，无需代理客户端（agentless），很多 Linux 发行版默认已安装了 Python 和 OpenSSH
- 安全性： 基于 OpenSSH
- 支持任务编排： 通过 playbook 来描述任务（使用非常简单的 YAML 语言）
- 幂等性（idempotent）： 一个任务执行 1 遍和执行 n 遍的结果是一样的，不会因为重复执行带来意外情况

## Project

### host

- c1(控制节点): 192.168.5.67
- c2: 192.168.5.133
- c3: 192.168.5.119

### pre-configure

```shell
# 控制节点: 修改主机名
[root@localhost-localdomain ~]# hostnamectl set-hostname c1

# 控制节点: 设置SSH密钥
# 创建 RSA 密钥对
[root@c1 ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:HRzWCFpeOmOQ9D58D+n95+XoH7FqxLb87EjYwCu5aHs root@c1
The key's randomart image is:
+---[RSA 2048]----+
|     .o.o.+o     |
|      .* =...    |
|      . B o      |
|       + +.o     |
|        S =o.  . |
|         +.+=+  o|
|         o.o*o.o.|
|       ..Eo .=+o+|
|      .oo.  .+*B+|
+----[SHA256]-----+

# 使用 ssh-copy-id 复制公钥复制到c2
[root@c1 ~]# ssh-copy-id root@192.168.5.133
The authenticity of host '192.168.5.133 (192.168.5.133)' can't be established.
ECDSA key fingerprint is SHA256:0vfeWV8kz8mR9/dRfQbIkeNqzgTKScDgT9LvH725M5I.
ECDSA key fingerprint is MD5:3f:2c:75:d5:29:9c:20:eb:2f:78:51:c8:a6:8c:66:e5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
rroot@192.168.5.133's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.5.133'"
and check to make sure that only the key(s) you wanted were added..

# 使用 ssh-copy-id 复制公钥复制到c3
[root@c1 ~]# ssh-copy-id root@192.168.5.119
The authenticity of host '192.168.5.119 (192.168.5.119)' can't be established.
ECDSA key fingerprint is SHA256:hUDazxPs1lgsqIBOlrOBXLjfnko9LMv74j6+efI0ZsU.
ECDSA key fingerprint is MD5:b3:d2:69:7a:e9:94:31:47:3c:ce:f2:f3:84:e5:65:2c.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.5.119's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.5.119'"
and check to make sure that only the key(s) you wanted were added.

# 使用 ssh-copy-id 复制公钥复制到c1
[root@c1 ~]# ssh-copy-id root@192.168.5.67
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.5.67's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.5.67'"
and check to make sure that only the key(s) you wanted were added.

# c1禁用密码验证
[root@c1 ~]# sed -i "s@PasswordAuthentication no@PasswordAuthentication yes@g" /etc/ssh/ssh_config 
[root@c1 ~]# systemctl restart sshd

# 使用 SSH 密钥对 c2 服务器进行身份验证, 修改主机名c2
[root@c1 ~]# ssh root@192.168.5.133
[root@localhost ~]# hostnamectl set-hostname c2

# 使用 SSH 密钥对 c3 服务器进行身份验证, 修改主机名c3
[root@c1 ~]# ssh root@192.168.5.119
[root@localhost ~]# hostnamectl set-hostname c3
```

### install

```shell
# 安装 Ansible
[root@c1 ~]# yum -y install ansible
Loaded plugins: fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Repository epel-debuginfo is listed more than once in the configuration
Repository epel-source is listed more than once in the configuration
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package ansible.noarch 0:2.9.27-1.el7 will be installed
--> Processing Dependency: python-httplib2 for package: ansible-2.9.27-1.el7.noarch
--> Processing Dependency: python-jinja2 for package: ansible-2.9.27-1.el7.noarch
--> Processing Dependency: python-paramiko for package: ansible-2.9.27-1.el7.noarch
--> Processing Dependency: python2-jmespath for package: ansible-2.9.27-1.el7.noarch
--> Processing Dependency: sshpass for package: ansible-2.9.27-1.el7.noarch
--> Running transaction check
---> Package python-jinja2.noarch 0:2.7.2-4.el7 will be installed
--> Processing Dependency: python-babel >= 0.8 for package: python-jinja2-2.7.2-4.el7.noarch
--> Processing Dependency: python-markupsafe for package: python-jinja2-2.7.2-4.el7.noarch
---> Package python-paramiko.noarch 0:2.1.1-9.el7 will be installed
---> Package python2-httplib2.noarch 0:0.18.1-3.el7 will be installed
---> Package python2-jmespath.noarch 0:0.9.4-2.el7 will be installed
---> Package sshpass.x86_64 0:1.06-2.el7 will be installed
--> Running transaction check
---> Package python-babel.noarch 0:0.9.6-8.el7 will be installed
---> Package python-markupsafe.x86_64 0:0.11-10.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                  Arch          Version             Repository     Size
================================================================================
Installing:
 ansible                  noarch        2.9.27-1.el7        epel           17 M
Installing for dependencies:
 python-babel             noarch        0.9.6-8.el7         base          1.4 M
 python-jinja2            noarch        2.7.2-4.el7         base          519 k
 python-markupsafe        x86_64        0.11-10.el7         base           25 k
 python-paramiko          noarch        2.1.1-9.el7         base          269 k
 python2-httplib2         noarch        0.18.1-3.el7        epel          125 k
 python2-jmespath         noarch        0.9.4-2.el7         epel           41 k
 sshpass                  x86_64        1.06-2.el7          extras         21 k

Transaction Summary
================================================================================
Install  1 Package (+7 Dependent packages)

Total download size: 19 M
Installed size: 113 M
Downloading packages:
(1/8): python-jinja2-2.7.2-4.el7.noarch.rpm                | 519 kB   00:05     
(2/8): python-markupsafe-0.11-10.el7.x86_64.rpm            |  25 kB   00:00     
(3/8): python-babel-0.9.6-8.el7.noarch.rpm                 | 1.4 MB   00:06     
(4/8): python-paramiko-2.1.1-9.el7.noarch.rpm              | 269 kB   00:00     
(5/8): ansible-2.9.27-1.el7.noarch.rpm                     |  17 MB   00:11     
(6/8): python2-jmespath-0.9.4-2.el7.noarch.rpm             |  41 kB   00:00     
(7/8): python2-httplib2-0.18.1-3.el7.noarch.rpm            | 125 kB   00:05     
(8/8): sshpass-1.06-2.el7.x86_64.rpm                       |  21 kB   00:05     
--------------------------------------------------------------------------------
Total                                              1.1 MB/s |  19 MB  00:17     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : sshpass-1.06-2.el7.x86_64                                    1/8 
  Installing : python2-httplib2-0.18.1-3.el7.noarch                         2/8 
  Installing : python-babel-0.9.6-8.el7.noarch                              3/8 
  Installing : python2-jmespath-0.9.4-2.el7.noarch                          4/8 
  Installing : python-paramiko-2.1.1-9.el7.noarch                           5/8 
  Installing : python-markupsafe-0.11-10.el7.x86_64                         6/8 
  Installing : python-jinja2-2.7.2-4.el7.noarch                             7/8 
  Installing : ansible-2.9.27-1.el7.noarch                                  8/8 
  Verifying  : python-markupsafe-0.11-10.el7.x86_64                         1/8 
  Verifying  : ansible-2.9.27-1.el7.noarch                                  2/8 
  Verifying  : python-paramiko-2.1.1-9.el7.noarch                           3/8 
  Verifying  : python2-jmespath-0.9.4-2.el7.noarch                          4/8 
  Verifying  : python-babel-0.9.6-8.el7.noarch                              5/8 
  Verifying  : python2-httplib2-0.18.1-3.el7.noarch                         6/8 
  Verifying  : sshpass-1.06-2.el7.x86_64                                    7/8 
  Verifying  : python-jinja2-2.7.2-4.el7.noarch                             8/8 

Installed:
  ansible.noarch 0:2.9.27-1.el7                                                 

Dependency Installed:
  python-babel.noarch 0:0.9.6-8.el7       python-jinja2.noarch 0:2.7.2-4.el7    
  python-markupsafe.x86_64 0:0.11-10.el7  python-paramiko.noarch 0:2.1.1-9.el7  
  python2-httplib2.noarch 0:0.18.1-3.el7  python2-jmespath.noarch 0:0.9.4-2.el7 
  sshpass.x86_64 0:1.06-2.el7            

Complete!

[root@c1 ~]# ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 07:36:10) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
```

### 设置库存文件

库存文件 包含有关您将使用 Ansible 管理的主机的信息。 您可以在清单文件中包含从一台到数百台服务器的任意位置，并且可以将主机组织成组和子组。 清单文件还经常用于设置仅对特定主机或组有效的变量，以便在剧本和模板中使用。

- /etc/ansible/ansible.cfg： 主配置文件
- /etc/ansible/hosts： 默认用来保存远程受管节点的清单文件
- /etc/ansible/roles/： 角色（role）目录

Ansible 使用的第一步必须提供受管节点的 主机清单（Host Inventory），默认为 /etc/ansible/hosts ，它是 INI 格式的配置文件，其注释信息写的很详细，我们可以参照示例来提供主机 IP、主机名，还可以将主机分组等（当然，你也可以创建新的主机清单文件，后续通过 ansible -i INVENTORY 指定新主机清单文件即可）。

```shell
[root@c1 ~]# cat << _AnsibleHost_ > /etc/ansible/hosts
[nfsservers]
c2 ansible_host=192.168.5.133
c3 ansible_host=192.168.5.119

[nisslaver]
c2 ansible_host=192.168.5.133

[nisclient]
c3 ansible_host=192.168.5.119

[all:vars]
ansible_python_interpreter=/usr/bin/python

_AnsibleHost_
```

每当您想检查库存时，都可以运行：

```shell
[root@c1 ~]# ansible-inventory --list -y
all:
  children:
    nisservers:
      hosts:
        c2:
          ansible_host: 192.168.5.133
          ansible_python_interpreter: /usr/bin/python
        c3:
          ansible_host: 192.168.5.119
          ansible_python_interpreter: /usr/bin/python
    ungrouped: {}
    vncservers:
      hosts:
        c2: {}
        c3: {}
```

### 测试连接

在设置清单文件以包含您的服务器之后，是时候检查 Ansible 是否能够连接到这些服务器并通过 SSH 运行命令。 

```shell
[root@c1 ~]# ansible all -m ping -u root
c2 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
c3 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

如果这是您第一次通过 SSH 连接到这些服务器，系统会要求您确认通过 Ansible 连接的主机的真实性。 出现提示时，键入 yes，然后点击 ENTER 进行确认。

一旦您从主机收到 "pong" 回复，这意味着您已准备好在该服务器上运行 Ansible 命令和剧本。 

### 运行临时命令

在确认您的 Ansible 控制节点能够与您的主机通信后，您可以开始在您的服务器上运行临时命令和 playbook。

您通常通过 SSH 在远程服务器上执行的任何命令都可以在清单文件中指定的服务器上使用 Ansible 运行。 例如，您可以使用以下命令检查所有服务器上的磁盘使用情况： 

```shell
[root@c1 ~]# ansible all -a "df -h" -u root
c2 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       113G  4.2G  109G   4% /
devtmpfs         15G     0   15G   0% /dev
tmpfs            15G     0   15G   0% /dev/shm
tmpfs            15G  9.9M   15G   1% /run
tmpfs            15G     0   15G   0% /sys/fs/cgroup
/dev/sda1       2.0G  166M  1.9G   9% /boot
tmpfs           3.0G   52K  3.0G   1% /run/user/0
c3 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       104G  4.2G   99G   5% /
devtmpfs         15G     0   15G   0% /dev
tmpfs            15G     0   15G   0% /dev/shm
tmpfs            15G  9.9M   15G   1% /run
tmpfs            15G     0   15G   0% /sys/fs/cgroup
/dev/sda1      1014M  166M  849M  17% /boot
tmpfs           3.0G   36K  3.0G   1% /run/user/0
```

### 常用命令

#### (1) ansible

ansible 命令是 Ansible 的主程序，用来执行 Ad-Hoc 临时命令，可参考章节 3.1。它的常用选项如下：

```shell
[root@CentOS ~]# ansible -h
Usage: ansible <host-pattern> [options]

Define and run a single task 'playbook' against a set of hosts

Options:
  -m MODULE_NAME, --module-name=MODULE_NAME
                        module name to execute (default=command)
  -a MODULE_ARGS, --args=MODULE_ARGS
                        module arguments
  -C, --check           don't make any changes; instead, try to predict some
                        of the changes that may occur
  -k, --ask-pass        ask for connection password
  -K, --ask-become-pass
                        ask for privilege escalation password
  -u REMOTE_USER, --user=REMOTE_USER
                        connect as this user (default=None)
  -b, --become          run operations with become (does not imply password
                        prompting)
  -i INVENTORY, --inventory=INVENTORY, --inventory-file=INVENTORY
                        specify inventory host path or comma separated host
                        list. --inventory-file is deprecated
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable
                        connection debugging)
```

其中 <host-pattern> 支持如下几种模式来匹配受管节点列表：

- `all`： 表示 主机清单（Host Inventory） 中的所有主机
- `*`： 通配符，比如 *、192.168.40.12*、*servers
- `:`： 或关系，比如 192.168.40.121:192.168.40.122、webservers:dbservers
- `:&`： 与关系，比如 webservers:&dbservers 表示在 webservers 组中且在 dbservers 组中，即 192.168.40.122 主机
- `:!`： 非关系，比如 webservers:!dbservers 表示在 webservers 组中但不在 dbservers 组中，即 192.168.40.121 主机

可以组合上述关系，或者使用正则表达式

#### (2) ansible-doc

ansible-doc 命令可以用来查看 模块 的帮助文档

```shell
# 1. 列出所有的内置模块
[root@CentOS ~]# ansible-doc -l
[root@CentOS ~]# ansible-doc -l | wc -l

# 2. 查看指定模块的帮助文档
[root@CentOS ~]# ansible-doc shell

# 3. 仅显示指定模块的选项参数在 playbook 中的示例用法
[root@CentOS ~]# ansible-doc -s shell
```

#### (3) ansible-galaxy

ansible-galaxy 命令会到 https://galaxy.ansible.com/ 去上传或下载 role，类似于 Docker 的镜像仓库

```shell
# 1. 安装 geerlingguy 用户所上传的 docker 这个角色
[root@CentOS ~]# ansible-galaxy install geerlingguy.docker
- downloading role 'docker', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-docker/archive/2.5.2.tar.gz
- extracting geerlingguy.docker to /root/.ansible/roles/geerlingguy.docker
- geerlingguy.docker (2.5.2) was installed successfully

# 2. 查看下载的 role 文件结构
[root@CentOS ~]# tree /root/.ansible/roles/geerlingguy.docker
/root/.ansible/roles/geerlingguy.docker
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── LICENSE
├── meta
│   └── main.yml
├── molecule
│   └── default
│       ├── molecule.yml
│       ├── playbook.yml
│       └── yaml-lint.yml
├── README.md
├── tasks
│   ├── docker-1809-shim.yml
│   ├── docker-compose.yml
│   ├── docker-users.yml
│   ├── main.yml
│   ├── setup-Debian.yml
│   └── setup-RedHat.yml
└── templates
    └── override.conf.j2

7 directories, 15 files

# 3. 查看已安装的 role 列表
[root@CentOS ~]# ansible-galaxy list
# /root/.ansible/roles
- geerlingguy.docker, 2.5.2
# /etc/ansible/roles
 [WARNING]: - the configured path /usr/share/ansible/roles does not exist.

# 4. 删除指定 role
[root@CentOS ~]# ansible-galaxy remove geerlingguy.docker
- successfully removed geerlingguy.docker
```

#### (4) ansible-playbook

ansible-playbook 命令用来执行编排后的任务列表（YAML 格式的 playbook 文件），类似于 docker-compose

将此剧本的内容下载到 Ansible 控制节点后，您可以使用 ansible-playbook 在库存中的一个或多个节点上执行它。 
以下命令将在默认清单文件中的 all 主机上执行 playbook，使用 SSH 密钥对身份验证以当前系统用户身份连接：

`ansible-playbook playbook.yml`

您还可以使用 -l 将执行限制为清单中的单个主机或一组主机：

`ansible-playbook -l host_or_group playbook.yml`

如果您需要指定不同的 SSH 用户来连接到远程服务器，您可以在该命令中包含参数 -u user：

`ansible-playbook -l host_or_group playbook.yml -u remote-user`

#### (5) ansible-vault

如果你不想让别人查看你的 playbook 文件内容，可以使用 ansible-vault 命令

## cluster

### pre

```shell
# 创建一个大约 600GB 左右的分割槽
[root@c1 ~]# fdisk /dev/sda
Command (m for help): `n`
Partition number (5-128, default 5): 
First sector (34-1953525134, default 348016640): 
Last sector, +sectors or +size{K,M,G,T,P} (348016640-1953525134, default 1953525134): `+600G`
Created partition 5

Command (m for help): `w`
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.

# 强制让核心重新捉一次 partition table
[root@c1 roles]# partprobe

# 磁盘格式化
[root@c1 /]# mkfs.xfs -f /dev/sda5
meta-data=/dev/sda5              isize=512    agcount=4, agsize=39321600 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=157286400, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=76800, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

# 创建挂载点
[root@c1 ~]# mkdir /public

# 挂载
[root@c1 /]# mount -f /dev/sda5 /public/

# 查阅硬盘内的相关信息
[root@c1 /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda4       150G   16G  135G  11% /
devtmpfs         15G     0   15G   0% /dev
tmpfs            15G   24M   15G   1% /dev/shm
tmpfs            15G   18M   15G   1% /run
tmpfs            15G     0   15G   0% /sys/fs/cgroup
/dev/sda2      1014M  157M  858M  16% /boot
/dev/sda1       200M  9.8M  191M   5% /boot/efi
tmpfs           3.0G   48K  3.0G   1% /run/user/0
/dev/sda5       600G   33M  600G   1% /public

# 设置主机名与IP的对应
[root@c1 ~]# cat << _ServerHost_ >> /etc/hosts
192.168.5.67  c1
192.168.5.133 c2
192.168.5.119 c3

_ServerHost_
```

### NFS

#### NFS Servers

```shell
# 配置分享的文件系统
[root@c1 ~]# echo "/public 192.168.5.133(rw,sync,no_root_squash) 192.168.5.119(rw,sync,no_root_squash)" >> /etc/exports
# 安装相关软件
[root@c1 ~]# yum install -y nfs-utils rpchbind
# 启动rpcbind服务并设置开机启动
[root@c1 ~]# systemctl start rpcbind && systemctl enable rpcbind
# 启动nfs服务并设置开机启动
[root@c1 ~]# systemctl start nfs && systemctl enable nfs
# 显示设定好的相关 exports 分享目录信息
[root@c1 ~]# showmount -e c1
```

#### NFS client

```shell
# write ansible-playbook yml file
[root@c1 ~]# vi nfs-client.yml
---
- hosts: all
  tasks:
    - name: disabled firewalld
      systemd: 
        name: firewalld
        state: stopped
        enabled: no

    - name: disabled selinux
      command: 'setenforce 0'
      ignore_errors: yes

    - name: create backup directory for yum repo
      file: 
        path: /etc/yum.repos.d/backup
        state: directory
    - name: replace yum repository
      shell: |
        mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/backup/
        curl -s http://mirrors.aliyun.com/repo/Centos-7.repo -o /etc/yum.repos.d/Centos-7.repo
        sed -i '/aliyuncs/d' /etc/yum.repos.d/Centos-7.repo
        curl -s http://mirrors.aliyun.com/repo/epel-7.repo -o /etc/yum.repos.d/epel-7.repo
        yum clean all
        yum makecache

    - name: install nfs and rpcbind
      yum: 
        name: "{{ packages }}"
        state: installed
      vars:
        packages:
        - rpcbind
        - nfs-utils

    - name: start and enable rpcbind service
      systemd: 
        name: rpcbind
        state: started
        enabled: yes
    - name: start and enable nfs service
      systemd: 
        name: nfs
        state: started
        enabled: yes

    - name: create shared directory
      file: 
        path: /public
        state: directory

    - name: mount server nfs directory
      mount:
        src: 192.168.5.67:/public
        path: /public
        fstype: nfs
        state: mounted

# execute ansible-playbook
[root@c1 ~]# ansible-playbook nfs-client.yml

PLAY [all] ***********************************************************************

TASK [Gathering Facts] ***********************************************************
ok: [c2]
ok: [c3]

TASK [disabled firewalld] ********************************************************
ok: [c2]
ok: [c3]

TASK [disabled selinux] **********************************************************
changed: [c3]
fatal: [c2]: FAILED! => {"changed": true, "cmd": ["setenforce", "0"], "delta": "0:00:00.004076", "end": "2024-09-10 16:59:07.411337", "msg": "non-zero return code", "rc": 1, "start": "2024-09-10 16:59:07.407261", "stderr": "setenforce: SELinux is disabled", "stderr_lines": ["setenforce: SELinux is disabled"], "stdout": "", "stdout_lines": []}
...ignoring

TASK [create backup directory for yum repo] **************************************
ok: [c3]
ok: [c2]

TASK [replace yum repository] ****************************************************
changed: [c2]
changed: [c3]

TASK [install nfs and rpcbind] ***************************************************
ok: [c2]
ok: [c3]

TASK [start and enable rpcbind service] ******************************************
ok: [c2]
ok: [c3]

TASK [start and enable nfs service] **********************************************
changed: [c3]
changed: [c2]

TASK [create shared directory] ***************************************************
changed: [c2]
changed: [c3]

TASK [mount server nfs directory] ************************************************
changed: [c3]
changed: [c2]

PLAY RECAP ***********************************************************************
c2                         : ok=10   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
c3                         : ok=10   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### NIS

#### NIS Servers

```shell
# 安装相关软件
[root@c1 root]# yum install yp-tools ypbind ypserv -y

# 设置NIS的域名(写入文件不会立即生效)
[root@c1 ~]# nisdomainname cluster
[root@c1 ~]# echo 'NISDOMAIN=centos' >> /etc/sysconfig/network

# 设置NIS服务器启动的端口号
[root@c1 ~]# echo 'YPSERV_ARGS=-p 1011' >> /etc/sysconfig/network
# 配置访问NIS服务器的权限
[root@c1 ~]# echo "192.168.5.67/255.255.255.0 :*:*:none" >> /etc/ypserv.conf
 
# 同步到slave servers
[root@c1 ~]# sed -i "s@NOPUSH=true@NOPUSH=false@g" /var/yp/Makefile
# 添加slave从服务器的主机名
[root@c1 ~]# cat << _YPServers_ > /var/yp/ypservers
c1
c2
_YPServers_

# 修改yppasswdd服务的端口，该功能与 NIS 客户端有关，允许用户在客户端修改服务器上的密码
[root@c1 ~]# sed -i "s@YPPASSWDD_ARGS=@YPPASSWDD_ARGS='--port 1012'@g" /etc/sysconfig/yppasswdd

# 启动yppasswdd服务并设置开机启动
[root@c1 ~]# systemctl start yppasswdd.service && systemctl enable yppasswdd.service
# 启动rpcbind服务并设置开机启动
[root@c1 ~]# systemctl start rpcbind.service && systemctl enable rpcbind.service
# 启动ypserv服务并设置开机启动
[root@c1 ~]# systemctl start ypserv.service && systemctl enable ypserv.service
# 启动ypxfrd服务并设置开机启动(slaver)
[root@c1 ~]# systemctl start ypxfrd.service && systemctl enable ypxfrd.service

# 修改添加用户默认参数, 修改默认家目录
[root@c1 ~]# sed -i "s@HOME=/home@HOME=/public/home@g" /etc/default/useradd
# 添加3个用户
[root@c1 ~]# useradd -u 1001 nisuser1
[root@c1 ~]# useradd -u 1002 nisuser2
[root@c1 ~]# useradd -u 1003 nisuser3
# 设置3个用户的初始密码
[root@c1 ~]# echo meimima | passwd --stdin nisuser1
[root@c1 ~]# echo meimima | passwd --stdin nisuser2
[root@c1 ~]# echo meimima | passwd --stdin nisuser3
# 转化帐号为数据库
[root@c1 ~]# /usr/lib64/yp/ypinit -m
	next host to add:  c1
	next host to add:  c2	# 从服务器的hostname
```

#### NIS Slaver

```shell
# write ansible-playbook yml file
[root@c1 ~]# vi nis-slaver.yml
---
- hosts: all
  tasks:
    - name: install nfs and rpcbind
      yum: 
        name: "{{ packages }}"
        state: installed
      vars:
        packages:
        - yp-tools
        - rpcbind
        - ypserv
        - ypbind

    - name: set hosts
      shell: |
        cat << _ServerHost_ >> /etc/hosts
        192.168.5.67  c1
        192.168.5.133 c2
        192.168.5.119 c3
        _ServerHost_

    - name: set nisdomain
      shell: |
        nisdomainname cluster
        echo 'NISDOMAIN=cluster' >> /etc/sysconfig/network

    - name: set nis access right
      shell: |
        echo '* : *: *: none' >> /etc/ypserv.conf

    - name: sync ypservices
      lineinfile:
        path: /var/yp/Makefile
        regex: '^NOPUSH=false'
        line: 'NOPUSH=true'

    - name: add to nisdomain
      lineinfile:
        dest: /etc/yp.conf
        insertafter: '^domain'
        line: "{{item}}"
      with_items:
        - "domain cluster server c1"
        - "domain cluster server c2"

    - name: revise authconfig
      lineinfile:
        dest: /etc/sysconfig/authconfig
        regex: '^USENIS=no'
        line: 'USENIS=yes'

    - name: setting for nsswitch
      lineinfile:
        dest: /etc/nsswitch.conf
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      with_items:
        - { regexp: "^passwd.*files sss", line: "passwd:     files nis sss" }
        - { regexp: "^shadow.*files sss", line: "shadow:     files nis sss" }
        - { regexp: "^group.*files sss", line: "group:     files nis sss" }
        - { regexp: "^hosts.*files dns", line: "hosts:      files nis dns myhostname" }

    - name: start and enable yppasswdd.service
      systemd: 
        name: yppasswdd
        state: started
        enabled: yes
    - name: start and enable rpcbind.service
      systemd: 
        name: rpcbind
        state: started
        enabled: yes
    - name: start and enable ypserv.service
      systemd: 
        name: ypserv
        state: started
        enabled: yes
    - name: start and enable ypbind.service
      systemd: 
        name: ypbind
        state: started
        enabled: yes

    - name: sync c1 database
      command: /usr/lib64/yp/ypinit -s c1

# execute ansible-playbook
[root@c1 ~]# ansible-playbook -l nisslaver nis-slaver.yml 
PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [c2]

TASK [install nfs and rpcbind] *************************************************
ok: [c2]

TASK [set hosts] ***************************************************************
changed: [c2]

TASK [set nisdomain] ***********************************************************
changed: [c2]

TASK [set nis access right] ****************************************************
changed: [c2]

TASK [sync ypservices] *********************************************************
ok: [c2]

TASK [add to nisdomain] ****************************************************************
changed: [c2] => (item=domain cluster server c1)
changed: [c2] => (item=domain cluster server c2)

TASK [revise authconfig] ***************************************************************
changed: [c2]

TASK [setting for nsswitch] ************************************************************
changed: [c2] => (item={u'regexp': u'^passwd.*files sss', u'line': u'passwd:     files nis sss'})
changed: [c2] => (item={u'regexp': u'^shadow.*files sss', u'line': u'shadow:     files nis sss'})
changed: [c2] => (item={u'regexp': u'^group.*files sss', u'line': u'group:     files nis sss'})
changed: [c2] => (item={u'regexp': u'^hosts.*files dns', u'line': u'hosts:      files nis dns myhostname'})

TASK [start and enable yppasswdd.service] **************************************
ok: [c2]

TASK [start and enable rpcbind.service] ****************************************
ok: [c2]

TASK [start and enable ypserv.service] *****************************************
ok: [c2]

TASK [start and enable ypbind.service] *************************************************
changed: [c2]

TASK [sync c1 database] ********************************************************
changed: [c2]

PLAY RECAP *********************************************************************
c2                         : ok=16   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

#### NIS client

```shell
[root@c1 ~]# vi nis-client.yml
# write ansible-playbook yml file
---
- hosts: all
  tasks:
    - name: install yp-tools and ypbind
      yum: 
        name: "{{ packages }}"
        state: installed
      vars:
        packages:
        - yp-tools
        - ypbind

    - name: set hosts
      shell: |
        cat << _ServerHost_ >> /etc/hosts
        192.168.5.67  c1
        192.168.5.133 c2
        192.168.5.119 c3
        _ServerHost_

    - name: set nisdomain
      shell: |
        nisdomainname cluster
        echo 'NISDOMAIN=cluster' >> /etc/sysconfig/network


    - name: add to nisdomain
      lineinfile:
        dest: /etc/yp.conf
        insertafter: '^domain'
        line: "{{item}}"
      with_items:
        - "domain cluster server c1"
        - "domain cluster server c2"

    - name: revise authconfig
      lineinfile:
        dest: /etc/sysconfig/authconfig
        regex: '^USENIS=no'
        line: 'USENIS=yes'

    - name: setting for nsswitch
      lineinfile:
        dest: /etc/nsswitch.conf
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      with_items:
        - { regexp: "^passwd.*files sss", line: "passwd:     files nis sss" }
        - { regexp: "^shadow.*files sss", line: "shadow:     files nis sss" }
        - { regexp: "^group.*files sss", line: "group:     files nis sss" }
        - { regexp: "^hosts.*files dns", line: "hosts:      files nis dns myhostname" }

    - name: start and enable rpcbind.service
      systemd: 
        name: rpcbind
        state: started
        enabled: yes
    - name: start and enable ypbind.service
      systemd: 
        name: ypbind
        state: started
        enabled: yes

# execute ansible-playbook
[root@c1 ~]# ansible-playbook -l nisclient nis-client.yml 

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [c3]

TASK [install yp-tools and ypbind] *********************************************
changed: [c3]

TASK [set hosts] ***************************************************************
changed: [c3]

TASK [set nisdomain] ***********************************************************
changed: [c3]

TASK [add to nisdomain] ****************************************************************
changed: [c3] => (item=domain cluster server c1)
changed: [c3] => (item=domain cluster server c2)

TASK [revise authconfig] *******************************************************
changed: [c3]

TASK [setting for nsswitch] ****************************************************
changed: [c3] => (item={u'regexp': u'^passwd.*files sss', u'line': u'passwd:     files nis sss'})
changed: [c3] => (item={u'regexp': u'^shadow.*files sss', u'line': u'shadow:     files nis sss'})
changed: [c3] => (item={u'regexp': u'^group.*files sss', u'line': u'group:     files nis sss'})
changed: [c3] => (item={u'regexp': u'^hosts.*files dns', u'line': u'hosts:      files nis dns myhostname'})

TASK [start and enable rpcbind.service] ****************************************
ok: [c3]

TASK [start and enable ypbind.service] *****************************************
changed: [c3]

PLAY RECAP *********************************************************************
c3                         : ok=7    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

### Slurm

参考[ansible-playbooks](https://github.com/chuanweihu/ansible-playbooks)

## Additional Resources

### Documentation

1. [Ansible 基础知识入门](https://www.redhat.com/zh/topics/automation/learning-ansible-tutorial)
2. [Ansible中文权威指南](http://www.ansible.com.cn/docs/)
3. [第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/basic-learning-16.html)

### Useful Websites

1. [CentOS KickStart 无人值守安装及自动部署ks脚本](https://www.cnblogs.com/hukey/p/14919346.html)
2. [在CentOS7.6上安装自动化运维工具Ansible以及playbook案例实操](https://www.cnblogs.com/Python-K8S/p/13226237.html)
3. [在CentOS 7上安装和配置Ansible的方法](https://developer.aliyun.com/article/1586452)
4. [centos7安装与配置ansible](https://www.cnblogs.com/soymilk2019/p/11173203.html)
5. [Ansible自动化运维｜第1章：简介](https://madmalls.com/blog/post/ansible-quickstart/)
6. [如何在Ubuntu18.04上设置SSH密钥](https://cainiaojiaocheng.com/How-to-set-up-ssh-keys-on-ubuntu-1804)
7. [如何在Ubuntu18.04上安装和配置Ansible](https://cainiaojiaocheng.com/How-to-install-and-configure-ansible-on-ubuntu-18-04)
8. [配置管理101：编写AnsiblePlaybook](https://cainiaojiaocheng.com/Configuration-management-101-writing-ansible-playbooks)
9. [playbook 剧本](https://curder.github.io/ansible-study/guide/playbook.html)
10. [ansible 剧本部署配置nfs](https://www.cnblogs.com/hekang520/p/15628616.html)

### Related Books

1. 鸟哥. 鸟哥的 Linux 私房菜: 基础学习篇. 人民邮电出版社, 2018.
2. 鸟哥. 鸟哥的 Linux 私房菜: 服务器架设篇. 机械工业出版社, 2008.
