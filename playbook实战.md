# 批量实现ssh免密
比如现在要实现的是ansible服务器可以免密连接其他服务器  
在ansible服务器上做如下操作

1、ansible里主机组的配置如下：  
```
[test]
172.20.125.181
[test:vars]
ansible_ssh_user='root'
ansible_ssh_pass='Qn@9865321'
ansible_ssh_port='22'
```
如果这里没配置密码，执行playbook的时候需要-k  
[root@iZ8v~]# ansible-playbook ssh.yml -k
SSH password: 

2、ssh-keygen 生成密钥
3、cat id_rsa.pub > authorized_keys

playbook的内容如下：
```
- hosts: test
  gather_facts: no
  tasks:
   - name: close ssh yes/no check
     lineinfile: path=/etc/ssh/ssh_config regexp='(.*)StrictHostKeyChecking(.*)' line="StrictHostKeyChecking no"
   - name: create .ssh directory
     file: dest=/root/.ssh mode=0600 state=directory
   - name: copy authorized_keys to client
     copy: src=/root/.ssh/authorized_keys dest=/root/.ssh/ mode=0600
```
