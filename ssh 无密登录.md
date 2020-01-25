# ssh 无密登录

## 1. 客户端

### 1.1 生成私钥

参考 [github生成 SSH keys](https://help.github.com/articles/connecting-to-github-with-ssh/) 

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### 1.2 设置秘钥

**很重要**的一点

`~/.ssh` 和 `～/.ssh/id_rsa` 的权限要限制下，否则不会被使用

```bash
# 限制 ～/.ssh 和 ～/.ssh/id_rsa 的权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa

# 使用如下命令将 ~/.ssh/id_rsa 加入 ssh-agent
ssh-add

# 使用如下命令查看已加入 ssh-agent 的私钥
ssh-add -l
```

### 1.3 上传公钥

将`～/.ssh/id_rsa.pub` 上传到服务端

```bash
scp -p port ~/.ssh/id_rsa.pub user@remotehost:/home/user/id_rsa.pub
```



## 2. 服务端

### 2.1 将公钥加入授权

```bash
# 确认是否有 /home/user/.ssh 文件夹存在，不存在则创建
mkdir .ssh

# 将上传的 /home/user/id_rsa.pub 增量加入 authorized_keys 文件
cat /home/user/id_rsa.pub >> /home/user/.ssh/authorized_keys
```

### 2.2 更改 ssh_config 设置

```bash
# 更改 /etc/ssh/ssh_config ,在 Host * 后加入下面的设置，或者放开其对应注释
Host *
	RSAAuthentication yes
    PubkeyAuthentication yes
    AuthorizedKeysFile /home/user/.ssh/authorized_keys
    
# 重启 sshd
service sshd restart
```



## 3. 客户端便捷设置

```bash
# 在 ~/.ssh 目录下新建 config 文件用于保存服务端信息
touch ~/.ssh/config

# 编辑其内容，形如
Host server1
    HostName 1.1.1.1
    Port 22
    User root

Host server2
    HostName 2.2.2.2
    Port 22
    User root

# 使用方式
ssh server1
```



