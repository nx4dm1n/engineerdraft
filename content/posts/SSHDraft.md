+++
date = "2014-11-23T11:47:25+08:00"
draft = false
title = "SSHDraft"

+++

## Black Magic Of SSH

### 绑定本地端口( Dynamic Port Forwarding)
其实本质上就是一个 SOCKS5 proxy  
ssh 通过加密传输数据, 特别适合**国情, 加入我们要让8080 端口的数据, 全部通过 SSH  加密转发到远程主机上

	ssh  -D 8080 user@host
	
### 遇到错误
	Bad dynamic forwarding specification
原因: ssh -D user@host 忘记了指定端口
	
#### 应用场景  
在咖啡厅等公共场所, 为了上网安全, 可以使用 ssh动态转发来保证安全.

### 本地端口转发( Local Port Forwarding)
#### 背景
公司从安全性上考虑, 通常会隔离网络, 不允许直接访问, 简单拓扑如下:   

		      host2
	host1 -->  X  --> host3
	
host1 无法直接访问 host3, host1 可以访问 host2, host2 可以访问 host3.   
可以通过 ssh 的 **本地端口转发** 来进行  

	ssh -L 2121:host3:21 user@host2
	
<!--more-->

**本地端口转发** 三个参数值表示:  本机绑定端口:目标主机:目标主机端口,  其中**本机绑定端口**意思是会在本机启动一个**本地监听的端口**.    
上面的命令实现的效果就是, 我们只要连接上 host1的2121 端口, 就等于连接上了 host2的21端口.  

	ftp localhost:2121
	
**本地端口转发** 让 host1 和 host2 直接形成 **SSH 隧道**   
注意其中**目标主机**的机器名或者 ip 是香港 host3 而言的.    

	ssh -L 5900:localhost:5900  host2
	
表示将本机的5900端口绑定到 host3的5900 端口. localhost 是想对 host2 而言.  

本地端口转发的意思是:  绑定本地机器端口, 进行数据转发.  


### 远程端口转发(Remote Port Forwarding)
远程端口转发: 对应就是 绑定远程机器端口, 进行数据转发.
#### 背景  
公司又从安全性上考虑, host2 通过 NAT 出去可以访问 host1,  host1 机器无法访问 host2, 同时 host2 和 host3 机器间能够相互访问.   我们希望能够 host1 直接访问 host3


	host1 <- host2
		   |     |
		   X     |
		   |     |
		   +---host3
		  
解决的办法是:   既然 host2 可以连接 host1, 那么就从 host2上建立也 host1的连接, 然后在 host1上使用这个连接即可.    
在 host2上执行

		ssh -R  2121:host3:21 host1
		
远程端口转发三个参数含义:  远程机器绑定端口: 目标主机:目标主机(Listen) 端口. 上面命令的意思是: 让 host1监听自己的2121 端口, 然后将所有的数据, 从 host2 转发到 host3 的21端口.    
绑定之后, 可以在 host1 上直连 host3.  

	ftp localhost:2121
	
##### 注意点
1.  2121 远程机器端口会在 ssh 运行之后, 自动在 host1 机器上启动.
2.  localhost 是针对host2 而言. 对比 Local 和 Remote 本质上 **localhost** 是针对 proxy 角色而言的,  在 local forwarding proxy 是:  host2,  remote 的 proxy 也是 host2
	
#### 典型应用场景
你在本机上开发了一个 web (9000端口) , 然后你想向你的朋友演示, 但是你没有外网 ip, 所有不可能直接访问.    
幸运的是你有一个外网可以访问的 VPS(3000端口), 这时候的解决方案是:
	
		ssh -R 3000:loclhost:9000 user@vps
默认情况下, ssh 是不会允许远程机器来进行端口转发.  需要在 **host1**上增加一行配置, 

	
		vim /etc/ssh/sshd_config
		GatewayPorts yes
		sudo service ssh restart 
		
### SSH Agent Forwarding
#### 典型应用场景
本机 laptop 和 github.com 建立了信任关系, 可以直接 ssh 上去,  然后登陆自己的 vps, 希望能够在 vps 上登陆 github.com , 正常情况是把 vps 的 public key 放到 github.com 但是这样有点麻烦, 可以直接用下面: 

	ssh -A  vpsserver  #会把本地的 ssh private key 带过去
	vpsservr$  ssh github.com
	
### SSH 别名
#### 典型应用场景  
自己手上有几台机器, 名字都比较长, 可以采用 ssh 别名来进行.  
`~/.ssh/config`   

	Host AliasName
	Hostname bastin.east.doc.example.com

### SSH 嵌套

	ssh -t hostA  ssh -t hostB ssh -t hostC
		
### SSH 有用参数

	-N Do not execute a remote command.  This is useful for just forwarding
             ports (protocol version 2 only).
	-T 不为这个连接分配 TTY
	-f      Requests ssh to go to background just before command execution.  This
             is useful if ssh is going to ask for passwords or passphrases, but
             the user wants it in the background.
             表示SSH连接成功后，转入后台运行。这样一来，你就可以在不中断SSH连接的情况下，在本地shell中执行其他操作。
             
    -g      Allows remote hosts to connect to local forwarded ports. 
    
    -n      Redirects stdin from /dev/null (actually, prevents reading from stdin).  This must be used when ssh is run in the background.  A common trick
             is to use this to run X11 programs on a remote machine.  For example, ssh -n shadows.cs.hut.fi emacs & will start an emacs on shad-
             ows.cs.hut.fi, and the X11 connection will be automatically forwarded over an encrypted channel.  The ssh program will be put in the back-
             ground.  (This does not work if ssh needs to ask for a password or passphrase; see also the -f option.)
     -o "StrictHostKeyChecking no"  取消机器 fingerprint 检查, 避免输入 yes/no 交互
             
比较常见选项组合:   
	
	-nNT

### Rsync 指定端口  
#### 背景
需要从服务器A(10086)向服务器 B(22端口) rsync 数据, 处于安全性上考虑, 服务器 A 的 ssh 隐藏为其他端口(10086 为例), 如果直接 rsync 会提示 rsync 直接采用10086 端口  
无法进行数据同步.  解决方法:  

    rsync -HavP -e 'ssh -p 22' username@hostname:src dst
	

#### Reference
[SSH Tunnel - Local and Remote Port Forwarding Explained With Examples](http://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html)

### SSH 指定端口
背景是为了安全起见, 通常会将 ssh 端口修改为非22端口, 而在使用 git 的时候, 通常是 ssh 端口, 常见的解决方法是修改 .git/conf 下调整为 

    ssh://git@domain.com:22/~/Projects/p1.git

每次都需要修改比较烦人, 比较简洁的方案是: 修改`.ssh/config` 文件, 其中 `foo.com` 是 git server 的域名

    Host foo.com
        HostName foo.com
        Port 22
        GSSAPIAuthentication no

### ssh-copy-id 添加信任关系  
1: 利用 ssh-keygen 生成一对密钥对   
2: ssh-copy-id  **-i**  user@host  
-i 默认是读取`~/.ssh/id_rsa.pub` 如果需要读取其它公钥文件, 只需要 -i  path2publickey 

### ssh 验证 public 和 private key 是否是一对
有时候在追查问题的时候, 我们验证是否因为 copy/paste 错误导致的问题.   
解决方案:  

	ssh-keygen -y -f <private key> 
	
	-y      This option will read a private OpenSSH format file and print an OpenSSH public key to stdout.
	
通过读取 private key 得到 public key , 然后直接对比 public 可以 即可

### 获取机器的fingerprint 
#### 背景
最近在折腾[sshpiper](https://github.com/tg123/sshpiper) 的时候遇到了一个诡异的问题, 需要判断 sshpiper 连接到的是哪台机器上.  在第一次连接的时候 ssh 会显示连接机器的 fingerprint, 因此需要通过机器的 fingerprint 来确定对应的机器.   
#### 获取方法 

	ssh-keygen -l -f <privatekey>    
	
	
### SSH 取消密码登陆   
#### 背景   
在运维的时候, 通常希望能够批量自动化执行命令, 可是经常会遇到有些机器需要使用密码登陆(忘记添加信任关系),  针对这些机器我们通常希望能够自动跳过.    
#### 解决方案  
    
    -o 
    
     PasswordAuthentication yes  
     
#### 其它相关参数

     StrictHostKeyChecking no   不进行 ssh key 验证ssh将不会自动把计算机的密匙加入"$HOME/.ssh/known_hosts"文件