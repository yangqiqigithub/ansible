# inventory作用
作用：通常用于定义要管理主机的认证信息，例如ssh登录用户名，密码等相关信息  
缺省文件：/etc/ansible/hosts  

# 主机组的定义
```
#不分组的主机放在文件最上方
green.example.com
blue.example.com
192.168.100.1
192.168.100.10

#主机组的定义
[webserver]
172.20.125.181
```
# 批量定义主机
```
#主机组的定义
[webserver]
172.20.125.[1:200]
```
# 内置参数
```
[webservers]
192.168.1.[31:32] ansible_ssh_user='root' ansible_ssh_pass='redhat'
```
# vars变量
```
[webservers]
192.168.1.[31:32]
[webservers:vars]
ansible_ssh_user='root'
ansible_ssh_pass='redhat'
ansible_ssh_port='22'
```
# 组继承
```
[nginx]
192.168.1.31
[apache]
192.168.1.32
[webservers:children]
apache
nginx
[webservers:vars]
ansible_ssh_user='root'
ansible_ssh_pass='redhat'
ansible_ssh_port='22'
```
# 自定义主机清单文件
1、修改配置文件，定义主机组目录  
ansible默认的主机组放在/etc/ansible/hosts这个文件里，修改配置文件，配置要存放主机清单文件的目录
```
/etc/ansible/ansible.cfg
inventory = /etc/ansible/hosts/
```
```
[root@iZ8vbe901lz5iykherzbihZ hosts]# pwd
/etc/ansible/hosts
[root@iZ8vbe901lz5iykherzbihZ hosts]# cat test |grep -v '^#' |grep -v '^$'
[test]
172.20.125.181
[root@iZ8vbe901lz5iykherzbihZ hosts]# cat webserver |grep -v '^#' |grep -v '^$'
[webserver]
172.20.125.181
[root@iZ8vbe901lz5iykherzbihZ hosts]# ansible webserver -m shell -a 'hostname' 还可以直接指定主机组
172.20.125.181 | CHANGED | rc=0 >>
ansible
[root@iZ8vbe901lz5iykherzbihZ hosts]# ansible -i /etc/ansible/hosts/webserver  webserver -m shell -a 'hostname' 可以指定主机组文件 主机组
172.20.125.181 | CHANGED | rc=0 >>
ansible
[root@iZ8vbe901lz5iykherzbihZ hosts]# 
```

# 动态获取主机清单
要求：  
1、输出的内容必须为json格式  
2、必须支持以下两个参数  
--list 列出所有主机  
--host ip 列出某个主机信息  
3、对语言没有限制  
python举例： test.py
chmod +x test.py 需要有可执行权限
```
#!/usr/bin/env python
#_*_coding:utf-8_*_
'''
以上两行的声明是必须的
'''
import json
import sys

def all():
    '''
    列出所有主机 必须是json
    :return: 
    '''
    info_dict = {
    "all":[
        "172.20.125.181",
        "172.20.125.181"]
    }
    print(json.dumps(info_dict,indent=4))

def group():
    '''
    定义主机组
    :return: 
    '''
    host1 = ['172.20.125.181']
    host2 = ['172.20.125.181','172.20.125.181']
    group1 = 'test1'
    group2 = 'test2'
    hostdata = {
        group1:{"hosts":host1},
        group2:{"hosts":host2}
    }
    print(json.dumps(hostdata,indent=4))

def host(ip):
    '''
    列出某个主机的信息
    :param ip: 
    :return: 
    '''
    info_dict = {
        "172.20.125.181": {
            "ansible_ssh_host":"172.20.125.181",
            "ansible_ssh_port":22,
            "ansible_ssh_user":"root",
            "ansible_ssh_pass":"123457"
        },
        "172.20.125.181": {
            "ansible_ssh_host":"172.20.125.181",
            "ansible_ssh_port":22,
            "ansible_ssh_user":"root",
        }
    }
    print(json.dumps(info_dict,indent=4))

if len(sys.argv) == 2 and (sys.argv[1] == '--list'):
    group()
elif len(sys.argv) == 3 and (sys.argv[1] == '--host'):
    host(sys.argv[2])
else:
    print("Usage: %s --list or --host <hostname>" % sys.argv[0])
    sys.exit(1)
```
执行结果如下：
```
[root@iZ8vbe901lz5iykherzbihZ ansible]# /etc/ansible/test.py --list
{
    "test1": {
        "hosts": [
            "172.20.125.181"
        ]
    }, 
    "test2": {
        "hosts": [
            "172.20.125.181", 
            "172.20.125.181"
        ]
    }
}
[root@iZ8vbe901lz5iykherzbihZ ansible]# /etc/ansible/test.py --host 172.20.125.181
{
    "172.20.125.181": {
        "ansible_ssh_host": "172.20.125.181", 
        "ansible_ssh_port": 22, 
        "ansible_ssh_user": "root"
    }
}
[root@iZ8vbe901lz5iykherzbihZ ansible]# ansible -i test.py all -m shell -a 'hostname'
172.20.125.181 | CHANGED | rc=0 >>
ansible
[root@iZ8vbe901lz5iykherzbihZ ansible]# ansible -i test.py test1 -m shell -a 'hostname'
172.20.125.181 | CHANGED | rc=0 >>
ansible
[root@iZ8vbe901lz5iykherzbihZ ansible]# ansible -i test.py test2 -m shell -a 'hostname'
172.20.125.181 | CHANGED | rc=0 >>
ansible
[root@iZ8vbe901lz5iykherzbihZ ansible]# 
```
