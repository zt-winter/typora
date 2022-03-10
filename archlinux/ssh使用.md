# ssh使用

本地主机A需要远程连接到主机B，则A自己生成一对公私钥，然后将公钥复制到B:~/home/username/.ssh/authorized_keys中。

### 生成ssh-key

```
ssh-keygen -t rsa -C "email.address"
```

### ssh-key管理

```
/* 生成的秘钥一般默认保存为id_rsa，当进行ssh连接时，默认使用的秘钥为id_rsa，这时如果对应的秘钥是其他文件，则连接失败。
因此需要编辑文件~/home/zt/.ssh/config 对多个ssh-key管理 */
# vim ~/home/zt/.ssh/config
//连接域名
Host github.com
HostName github.com
//认证的方式，公钥认证
PreferredAuthentications publickey
//私钥文件位置
IdentityFile ~/.ssh/github_rsa

Host git.mesalab.cn
HostName git.mesalab.cn
PreferredAuthentications publickey
IdentityFile ~/.ssh/mesalab_rsa

# zt @ zt-zjh in ~/.ssh [11:35:28]
$ ssh -T git@git.mesalab.cn
Welcome to GitLab, @zhangtao!

$ ssh -T git@github.com
Hi zt-winter! You've successfully authenticated, but GitHub does not provide shell access.
```

