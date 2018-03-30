脚本修改密码:
```shell
passwd ubuntu<<EOF
ubuntu
ubuntu
EOF
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
service ssh restart
```
