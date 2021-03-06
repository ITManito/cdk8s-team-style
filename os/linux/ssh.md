# SSH（Secure Shell）介绍

## 修改完记得重启服务

- CentOS 7 命令：`systemctl restart sshd.service`

## SSH 修改连接端口

- 配置文件介绍（记得先备份）：`sudo vim /etc/ssh/sshd_config`
- 打开这一行注释：Port 22
	- 自定义端口选择建议在万位的端口，如：10000-65535之间，假设这里我改为 60001
- CentOS 7：添加端口：`firewall-cmd --zone=public --add-port=60001/tcp --permanent`
	- 重启防火墙：`firewall-cmd --reload`

## 设置超时

- ClientAliveInterval指定了服务器端向客户端请求消息的时间间隔, 默认是0，不发送。而ClientAliveInterval 300表示5分钟发送一次，然后客户端响应，这样就保持长连接了。
- ClientAliveCountMax，默认值3。ClientAliveCountMax表示服务器发出请求后客户端没有响应的次数达到一定值，就自动断开，正常情况下，客户端不会不响应。
- 正常我们可以设置为：
    - TCPKeepAlive yes
	- ClientAliveInterval 300
	- ClientAliveCountMax 3
	- 按上面的配置的话，300*3＝900秒＝15分钟，即15分钟客户端不响应时，ssh连接会自动退出。

## SSH 允许 root 账户登录

- 编辑配置文件（记得先备份）：`sudo vim /etc/ssh/sshd_config`
 - 允许 root 账号登录
    - 注释掉：`PermitRootLogin without-password`
    - 新增一行：`PermitRootLogin yes`

## SSH 不允许 root 账户登录

- 新增用户和把新增的用户改为跟 root 同等权限方法：[Bash.md]
- 编辑配置文件（记得先备份）：`sudo vim /etc/ssh/sshd_config`
    - 注释掉这一句（如果没有这一句则不管它）：`PermitRootLogin yes`

## SSH 密钥登录

- 生成秘钥和公钥文件，命令：`sudo ssh-keygen`，在交互提示中连续按三次回车，如果看得懂交互的表达，那就根据你自己需求来。默认生成密钥和公钥文件是在：/root/.ssh。
- 进入生成目录：`cd /root/.ssh`，可以看到有两个文件：id_rsa (私钥) 和 id_rsa.pub (公钥)
- 在 .ssh 目录下创建 SSH 认证文件，命令：`touch /root/.ssh/authorized_keys`
- 将公钥内容写到SSH认证文件里面，命令：`cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys`
- 修改SSH认证文件权限，命令：
   - `sudo chmod 700 /root/.ssh`
   - `sudo chmod 600 /root/.ssh/authorized_keys`
- 重启服务：`sudo service sshd restart`
- 设置 SSH 服务默认启动：`sudo sysv-rc-conf ssh on`
- 现在 SSH 客户端可以去拿着 SSH 服务器端上的 id_rsa，在客户端指定秘钥文件地址即可，这个一般由于你使用的客户端决定的，我这里推荐的是 Xshell 软件。
- 另外一种方法可以查看：[SSH 免密登录（推荐）](SSH-login-without-password.md)

## 限制只有某一个IP才能远程登录服务器

- 在该配置文件：`vim /etc/hosts.deny`
	- 添加该内容：`sshd:ALL`
- 在该配置文件：`vim /etc/hosts.allow`
	- 添加该内容：`sshd:123.23.1.23`

## 限制某些用户可以 SSH 访问

- 在该配置文件：`vim /etc/ssh/sshd_config`
	- 添加该内容：`AllowUsers root userName1 userName2`


## 常用 SSH 连接终端

- Windows -- Xshell：<http://www.youmeek.com/ssh-terminal-emulator-recommend-xshell-and-xftp/>
- Mac -- ZOC：<http://xclient.info/s/zoc-terminal.html>

## 查看 SSH 登录日志

#### CentOS 6

- 查看登录失败记录：`cat /var/log/auth.log | grep "Failed password"`
	- 如果数据太多可以用命令：`tail -500f /var/log/auth.log | grep "Failed password"`
- 统计哪些 IP 尝试多少次失败登录记录：`grep "Failed password" /var/log/auth.log | awk ‘{print $11}’ | uniq -c | sort -nr`
- 统计哪些 IP 尝试多少次失败登录记录，并排序：`grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | more`


#### CentOS 7

- 查看登录失败：`egrep "Failed|Failure" /var/log/secure`，可以得到类似信息：
	- 如果数据太多，可以用命令：`tail -500f /var/log/secure | egrep "Failed|Failure"`

```
Feb 21 16:10:15 U5NHTIHOW67HKAH sshd[32753]: Failed password for invalid user oracle from 1.175.83.6 port 46560 ssh2
Feb 21 16:10:16 U5NHTIHOW67HKAH sshd[32750]: Failed password for root from 42.7.26.88 port 62468 ssh2
Feb 21 16:10:17 U5NHTIHOW67HKAH sshd[32744]: Failed password for root from 42.7.26.85 port 36086 ssh2
Feb 21 16:10:18 U5NHTIHOW67HKAH sshd[32756]: Failed password for invalid user oracle from 1.175.83.6 port 46671 ssh2
Feb 21 16:10:20 U5NHTIHOW67HKAH sshd[32744]: Failed password for root from 42.7.26.85 port 36086 ssh2
Feb 21 16:10:21 U5NHTIHOW67HKAH sshd[32750]: Failed password for root from 42.7.26.88 port 62468 ssh2
Feb 21 16:10:21 U5NHTIHOW67HKAH sshd[32758]: Failed password for invalid user oracle from 1.175.83.6 port 46811 ssh2
```

- 查看登录失败统计：`grep "authentication failure" /var/log/secure`，可以得到类似信息：
	- 如果数据太多，可以用命令：`tail -500f /var/log/secure | grep "authentication failure"`

```
Feb 21 02:01:46 U5NHTIHOW67HKAH sshd[16854]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:01:46 U5NHTIHOW67HKAH sshd[16854]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:01:47 U5NHTIHOW67HKAH sshd[16858]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:02:02 U5NHTIHOW67HKAH sshd[16858]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:02:02 U5NHTIHOW67HKAH sshd[16858]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:03:11 U5NHTIHOW67HKAH sshd[16870]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:03:25 U5NHTIHOW67HKAH sshd[16870]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:03:25 U5NHTIHOW67HKAH sshd[16870]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:03:29 U5NHTIHOW67HKAH sshd[16872]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:03:44 U5NHTIHOW67HKAH sshd[16872]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:03:44 U5NHTIHOW67HKAH sshd[16872]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:03:45 U5NHTIHOW67HKAH sshd[16875]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:04:01 U5NHTIHOW67HKAH sshd[16875]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:04:01 U5NHTIHOW67HKAH sshd[16875]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:04:05 U5NHTIHOW67HKAH sshd[16878]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:04:20 U5NHTIHOW67HKAH sshd[16878]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:04:20 U5NHTIHOW67HKAH sshd[16878]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:04:24 U5NHTIHOW67HKAH sshd[16883]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:04:40 U5NHTIHOW67HKAH sshd[16883]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:04:40 U5NHTIHOW67HKAH sshd[16883]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:04:43 U5NHTIHOW67HKAH sshd[16886]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:04:59 U5NHTIHOW67HKAH sshd[16886]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:04:59 U5NHTIHOW67HKAH sshd[16886]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:05:02 U5NHTIHOW67HKAH sshd[16888]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:05:08 U5NHTIHOW67HKAH sshd[16891]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=146.0.228.146
Feb 21 02:05:18 U5NHTIHOW67HKAH sshd[16888]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:05:18 U5NHTIHOW67HKAH sshd[16888]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:05:20 U5NHTIHOW67HKAH sshd[16899]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:05:34 U5NHTIHOW67HKAH sshd[16899]: Disconnecting: Too many authentication failures for root [preauth]
Feb 21 02:05:34 U5NHTIHOW67HKAH sshd[16899]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:05:37 U5NHTIHOW67HKAH sshd[16901]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=42.7.26.85  user=root
Feb 21 02:05:53 U5NHTIHOW67HKAH sshd[16901]: Disconnecting: Too many authentication failures for root [preauth]
```

## 防止 SSH 暴力破解：DenyHosts

- 官网地址：<https://github.com/denyhosts/denyhosts>
- 参考文章：
	- <http://blog.51cto.com/linuxroad/673425>
	- <http://blog.csdn.net/wanglei_storage/article/details/50849070>
	- <https://chegva.com/2338.html>
	- <http://blog.csdn.net/miner_k/article/details/78948100>

-------------------------------------------------------------------


## SSH 免密登录

### 关键点

- 免密登录的关键点在于理解谁登录谁。
- A 生成的公钥给 B，也给 C、D，则 A 可以直接免密 SSH 登录 B、C、D

### A 生成密钥

- 在 A 机器上输入命令：`ssh-keygen`
	- 根据提示回车，共有三次交互提示，都回车即可。
- 生成的密钥目录在：**/root/.ssh**
- 写入：`cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys`
- 测试：`ssh localhost`


### 把 A 的公钥发给 B

- 假设 B 机器的 ip：`192.168.1.105`
- 则在 A 机器上输入：`ssh-copy-id -i /root/.ssh/id_rsa.pub -p 22 root@192.168.1.105`，根据提示输入 B 机器的 root 密码，成功会有相应提示

### 测试 A 免密登录到 B

- 在 A 机器上输入命令：`ssh -p 22 root@192.168.1.105`，则会相应登录成功的提示


### 如果是用 pem 登录的话，用 ssh-copy-id 是无法使用的

- 先保存 A 的 pub 到本地：`sz /root/.ssh/id_rsa.pub`
- 登录 B 机子：`cd /root/.ssh/`
- 如果 B 机子没有 authorized_keys 文件则创建：`touch /root/.ssh/authorized_keys`
	- 设置权限：`chmod 600 /root/.ssh/authorized_keys `
- 上传 pub 文件到 B 机子，并在 B 机子上执行：`cd /root/.ssh/ && cat id_rsa.pub >> authorized_keys`


-------------------------------------------------------------------



## SSH 资料

- <http://www.jikexueyuan.com/course/861_1.html?ss=1> 
- <http://www.361way.com/ssh-autologout/4679.html> 
- <http://www.osyunwei.com/archives/672.html> 
- <https://www.tecmint.com/find-failed-ssh-login-attempts-in-linux/> 
