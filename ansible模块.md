## 命令模块
### command
执行简单的命令，不支持"<"，">"，"|"，";"，"&"等符号  
参数：  
chdir		在执行命令前，进入到指定目录中  
creates		判断指定文件是否存在，如果存在，不执行后面的操作  
removes		判断指定文件是否存在，如果存在，执行后面的操作  
```
[qiqi@ansible ~]$ ansible test -m command -a 'hostname'
172.20.125.179 | CHANGED | rc=0 >>
iZ8vbe901lz5iux1yntggsZ
[qiqi@ansible ~]$ ansible test -m command -a 'chdir=/home/qiqi ls'
172.20.125.179 | CHANGED | rc=0 >>
aa
bb
cc
dd
[qiqi@ansible ~]$ ansible test -m command -a 'touch aa creates=/home/qiqi/aa'
172.20.125.179 | SUCCESS | rc=0 >>
skipped, since /home/qiqi/aa exists
[qiqi@ansible ~]$ ansible test -m command -a 'touch aa removes=/home/qiqi/aa'
[WARNING]: Consider using the file module with state=touch rather than running 'touch'.  If you need to use command because
file is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid
of this message.
172.20.125.179 | CHANGED | rc=0 >>
```
### shell
基本什么命令都能执行，支持"<"，">"，"|"，";"，"&"等符号  
参数：  
chdir		在执行命令前，进入到指定目录中  
creates		判断指定文件是否存在，如果存在，不执行后面的操作  
removes		判断指定文件是否存在，如果存在，执行后面的操作  
```
[qiqi@ansible ~]$ ansible test -m shell -a 'ls |grep aa'
172.20.125.179 | CHANGED | rc=0 >>
aa
[qiqi@ansible ~]$ ansible test -m shell -a 'touch aa creates=/home/qiqi/aa'
172.20.125.179 | SUCCESS | rc=0 >>
skipped, since /home/qiqi/aa exists
[qiqi@ansible ~]$ ansible test -m shell -a 'touch aa removes=/home/qiqi/aa'
[WARNING]: Consider using the file module with state=touch rather than running 'touch'.  If you need to use command because
file is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid
of this message.
172.20.125.179 | CHANGED | rc=0 >>
```
## 文件模块
### copy  
copy模块在复制数据时，如果数据为软链接文件，会将链接指定源文件进行复制  
参数：    
src  本地文件地址 绝对路径     
dest  远程节点地址 决定路径，无则添加有则覆盖    
backup	no* yes 默认是no，复制文件到远程节点无则添加有则覆盖；如果设置成yes，如果源文件和远程节点上的文件内容一致不会产生备份文件，否则会产生以时间结尾的备份文件   
content		在文件中添加信息，默认覆盖，如果添加了backup=yes，是先备份再覆盖       
group		文件数据复制到远程主机，设置文件属组用户信息     
mode		文件数据复制到远程主机，设置数据的权限 eg 0644 0755    
owner		文件数据复制到远程主机，设置文件属主用户信息     
```  
[qiqi@ansible ~]$ ansible test -m copy -a 'src=/home/qiqi/aa dest=/home/qiqi/'
172.20.125.179 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "460767d6a9c762aed4be93b6b80f856887bf0f81", 
    "dest": "/home/qiqi/aa", 
    "gid": 1000, 
    "group": "qiqi", 
    "md5sum": "a2bd5c36c598231c09fc3f51638f1d4e", 
    "mode": "0664", 
    "owner": "qiqi", 
    "size": 8, 
    "src": "/home/qiqi/.ansible/tmp/ansible-tmp-1604411839.99-1734-81111826310129/source", 
    "state": "file", 
    "uid": 1000
}
[qiqi@ansible ~]$ ansible test -m copy -a "src=/home/qiqi/bb dest=/home/qiqi/ backup=yes"
172.20.125.179 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "backup_file": "/home/qiqi/bb.4714.2020-11-03@22:05:04~", 
    "changed": true, 
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "dest": "/home/qiqi/bb", 
    "gid": 1000, 
    "group": "qiqi", 
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e", 
    "mode": "0664", 
    "owner": "qiqi", 
    "size": 0, 
    "src": "/home/qiqi/.ansible/tmp/ansible-tmp-1604412303.39-1947-100816268755545/source", 
    "state": "file", 
    "uid": 1000
}
[qiqi@ansible ~]$ ansible test -m copy -a "content=mmmmmm dest=/home/qiqi/aa backup=yes"
172.20.125.179 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "backup_file": "/home/qiqi/aa.4916.2020-11-03@22:08:37~", 
    "changed": true, 
    "checksum": "cecec3ec436bf58a4ecce3e179835e25ff691f3e", 
    "dest": "/home/qiqi/aa", 
    "gid": 1000, 
    "group": "qiqi", 
    "md5sum": "9aee390f19345028f03bb16c588550e1", 
    "mode": "0664", 
    "owner": "qiqi", 
    "size": 6, 
    "src": "/home/qiqi/.ansible/tmp/ansible-tmp-1604412516.79-1993-238377873990313/source", 
    "state": "file", 
    "uid": 1000
}
[root@ansible ansible]# ansible test -m copy -a "src=/home/qiqi/bb dest=/home/qiqi/ group=www owner=www mode=777"
172.20.125.179 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "dest": "/home/qiqi/bb", 
    "gid": 1001, 
    "group": "www", 
    "mode": "0777", 
    "owner": "www", 
    "path": "/home/qiqi/bb", 
    "size": 0, 
    "state": "file", 
    "uid": 1001
}
```
### fetch   
抓取远程节点的文件到管理机上，只能是文件，有则覆盖，递归抓取
参数：   
src 远程节点的文件地址，文件地址
dest 管理机上的存放抓取文件的目录，目录
```
[qiqi@ansible ~]$  ansible test -m fetch -a 'src=/home/qiqi/client dest=/home/qiqi/'
172.20.125.179 | CHANGED => {
    "changed": true, 
    "checksum": "0a792be8257d2fa952d2fdb0afb8ac4bb283ddd5", 
    "dest": "/home/qiqi/172.20.125.179/home/qiqi/client", 
    "md5sum": "1b9dbc9109c79037173fe990591ba615", 
    "remote_checksum": "0a792be8257d2fa952d2fdb0afb8ac4bb283ddd5", 
    "remote_md5sum": null
}
[qiqi@ansible ~]$ ls
172.20.125.179  aa  bb  cc  dd
[qiqi@ansible ~]$ tree 172.20.125.179/ 抓取到管理机的目录结构
172.20.125.179/
└── home
    └── qiqi
        └── client

2 directories, 1 file
```
### file  
对文件的基本操作，对文件或者目录的创建、删除、修改权限等     
参数：      
path 在远程节点创建的文件或目录路径，文件无则创建，有则更新时间戳；目录无则创建，有则不作为  
state 指定在远程节点创建的类型 touch-文件 directory-目录 hard/link-链接文件



    
