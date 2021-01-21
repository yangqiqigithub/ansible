# shell、command、raw 模块
命令模块

command 执行简单的命令，不支持"<"，">"，"|"，";"，"&"等符号  

shell 基本什么命令都能执行，支持"<"，">"，"|"，";"，"&"等符号,执行命令的时候使用的是/bin/sh，所以shell模块可以执行任何命令

raw 功能类似与前面说的command、shell能够完成的操作，raw也都能完成。不同的是，raw模块不需要远程主机上的python环境。

#### 选项
chdir 在执行命令前，进入到指定目录中  
creates 判断指定文件是否存在，如果存在，不执行后面的操作   
removes 判断指定文件是否存在，如果存在，执行后面的操作
#### shell.yml
```
- hosts: test 指定主机组或IP
  remote_user: root 远程执行用户
  tasks: 任务列表
    - name: ansible shell 任务描述自定义
      shell: ps -ef |grep ssd > /tmp/sshd.log && mkdir /tmp/test 模块名字: 命令，多个命令用&&连接
    - name: ansible command
      command: touch /tmp/commandtest
```
执行palybook   
```
[root@ansible]# ansible-playbook shell.yml 

PLAY [test] **********************************************
执行的主机组是test

TASK [Gathering Facts] **********************************************************
ok: [172.20.125.181] 默认任务收集远程主机信息，可在配置文件定义是否收集

TASK [ansible shell] 第一个任务的描述信息 **********************************************************
changed: [172.20.125.181] 第一个任务执行完毕

TASK [ansible command] 第二个任务的描述信息 **********************************************************
[WARNING]: Consider using the file module with state=touch rather than running 'touch'.  If you need to use command because
file is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid 
of this message.警告信息可通过设置文件文件command_warnings=False去掉
changed: [172.20.125.181] 第二个任务执行完毕

PLAY RECAP ***********************************************
172.20.125.181             : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
针对172.20.125.181这个主机，各个状态的任务的个数
```
# file模块  
file模块主要用于远程主机上的文件操作
#### 选项
force：需要在两种情况下强制创建软链接，一种是源文件不存在但之后会建立的情况下；另一种是目标软链接已存在,需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no   

group：定义文件/目录的属组   
mode：定义文件/目录的权限  
owner：定义文件/目录的属主  
path：必选项，定义文件/目录的路径  
recurse：递归的设置文件的属性，只对目录有效  
src：要被链接的源文件的路径，只应用于state=link的情况   
dest：被链接到的目标路径，只应用于state=link的情况   

state： 有如下几个选项：
	directory：表示目录，如果目录不存在，则创建目录。
	link：创建软链接
	hard：创建硬链接
	touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
	absent：删除目录、文件或者取消链接文件。

#### file.yml
```
- hosts: test
  remote_user: root
  tasks:
  - name: touch file
    file: path=/tmp/touchfile state=touch mode=0777 owner=qiqi group=qiqi
  - name: mkdir directory
    file: path=/tmp/testdir state=directory mode=0777 recurse=yes
  - name: ln -s
    file: src=/tmp/testdir dest=/tmp/testdest state=link
  - name: delete file
    file: path=/tmp/sshd.log state=absent
```
执行playbook
```
[root@ansible]# ansible-playbook file.yml 

PLAY [test] ******************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [172.20.125.181]

TASK [touch file] ************************************************************************************************************
changed: [172.20.125.181]

TASK [mkdir directory] *******************************************************************************************************
changed: [172.20.125.181]

TASK [ln -s] *****************************************************************************************************************
changed: [172.20.125.181]

TASK [delete file] ***********************************************************************************************************
changed: [172.20.125.181]

PLAY RECAP *******************************************************************************************************************
172.20.125.181             : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
# copy模块
Copy模块用于复制文件、目录到远程主机
#### 选项：  
backup：在覆盖之前将原文件备份，备份文件包含时间信息。有两个选项：yes|no。不是每一次都会备份，只有当文件发生改变时，才会备份，产生备份文件。 

dest：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录   

directory_mode：递归的设定目录的权限，默认为系统默认权限   
force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes

src：要复制到远程主机的文件在本地的地址，可以是绝对路径，也可以是相对路径。如果路径是一个目录，它将递归复制。在这种情况下，如果路径使用”/”来结尾，则只复制目录里的内容，如果没有使用”/”来结尾，则包含目录在内的整个内容全部复制，类似于rsync。

#### copy.yml
```
- hosts: test
  remote_user: root
  gather_facts: false
  tasks:
  - name: copy file chown file
    copy: src=/home/qiqi/copyfile dest=/home/qiqi/ owner=qiqi group=qiqi mode=0777 backup=yes
  - name: copy dir
    copy: src=/tmp/tmptest/ dest=/home/qiqi mode=0777 backup=yes
  - name: copy dir
    copy: src=/home/qiqi/test dest=/tmp/
  - name: check file
    copy: src=/etc/sudoers dest=/home/qiqi/ validate='visudo -cf %s'
```
validate 检测文件的方式，具体检测方式自定义
```
[root@ansible]# visudo -cf /etc/sudoers 检测sudoers文件的语法
/etc/sudoers: parsed OK
```
执行结果
```
[root@iZ8vb10vvaorjiql15sbfjZ qiqi]# ls
copyfile  copyfile.14645.2020-11-12@22:03:26~  sudoers  tmptest  tmptest.15059.2020-11-12@22:04:11~
```
# synchronize模块
针对大文件大目录拷贝，有压缩功能
#### 选项
compress:是否开启压缩，默认开启。 

copy_links:复制链接文件，默认为no，注意后面还有一个links参数。 

delete:表示删除管理机中没有但远程主机存在的文件，使两边内容一样，以管理机为主，默认为no。 

src：要复制到远程主机的文件在管理机上的路径。  
dest:要将文件复制到的远程主机的绝对路径。

dest_prot：默认为22端口，走ssh协议，表示远程主机端口。  
mode：可选push和pull模块，push模块的话，一般用于从管理机向远程主机上传文件，pull模式用于从远程主机上取文件到管理机。  

注意，使用这个模块的时候，必须保证远程主机上有rsync这个命令，不然会报错。  
#### synchronize.yml
```
- hosts: test
  remote_user: root
  tasks:
    - name: synchronize model test
      synchronize: src=/tmp/ dest=/home/qiqi/ delete=yes
```
*src=/tmp/是将tmp里的文件或目录拷贝走*  
*src=/tmp 是将tmp这个目录拷贝走*    
*delete 慎用*   

# unarchive模块
unarchive模块用来实现解压缩，也就是将压缩文件解压分发到远程不同节点上。
#### 选项
src: 源文件路径，这个源文件在管理机上  
dest: 指定远程主机的文件路径 
mode：设置远程主机上文件权限  
####  unarchive.yml
```
- hosts: test
  remote_user: root
  tasks:
  - name: unarchive test
    unarchive: src=/tmp/tmptest.tar.gz dest=/home/qiqi/
```
# service模块
用于管理远程主机上的服务
#### 选项
enabled：是否开机启动 yes|no  
name：必选项，服务名称   
pattern：定义一个模式，如果通过status指令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务依然在运行  
sleep：如果执行了restarted，在则stop和start之间沉睡几秒钟  
state：对当前服务执行启动，停止、重启、重新加载等操（started,stopped,restarted,reloaded）
#### service.yml
```
- hosts: test
  remote_user: root
  tasks:
  - name: service model test
    service: name=sshd state=restarted enabled=yes
```
# handler notify模块
handler和notify合作，实现任务触发
```
- hosts: 172.20.125.181  
  remote_user: root
  tasks:
  - name: install apache
    yum: name=httpd state=latest
  - name: install configure file for httpd
    copy: src=/root/conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
  - name: start httpd service
    service: enabled=true name=httpd state=started
    notify: 做完上述操作需要触发的事件名称，如果上述操作早在之前做完了，本次执行都是绿色，也不会触发下边的事件
    - check httpd
    - rm conf
  handlers:
   - name: check httpd
     shell: netstat -tunlp |grep 80
   - name: rm conf
     shell: rm -rf /root/conf/httpd.conf
```
# cron模块
用于管理计划任务
#### 选项
backup：对远程主机上的原任务计划内容修改之前做备份   
cron_file：用来指定一个计划任务文件，也就是将计划任务写到远程主机上/etc/cron.d目录下，创建一个文件对应的计划任务   
day：日（1-31，*，*/2,……）   
hour：小时（0-23，*，*/2，……）    
minute：分钟（0-59，*，*/2，……）   
month：月（1-12，*，*/2，……）   
weekday：周（0-7，*，……）  
job：要执行的任务，依赖于state=present   
name：定义定时任务的描述信息   
special_time： 特殊的时间范围，参数：reboot（重启时）,annually（每年）,monthly（每月）,weekly（每周）,daily（每天）,hourly（每小时）   
state：确认该任务计划是创建还是删除,有两个值可选，分别是present和absent，present表示创建定时任务，absent表示删除定时任务，默认为present。  
user：以哪个用户的身份执行job指定的任务。  

#### cron.yml
```
- hosts: test
  remote_user: root
  tasks:
  - name: create mk cron
    cron: backup=true name=mk minute=*/1  user=root job='/root/mk.sh' 
    要确保远程主机有有这个脚本
```
# yum模块
使用yum包管理器来管理软件包
#### 选项
config_file：yum的配置文件   
disable_gpg_check：关闭gpg_check   
disablerepo：不启用某个源   
enablerepo：启用某个源  
name：要进行操作的软件包的名字，也可以传递一个url或者一个本地的rpm包的路径   
state：表示要安装还是删除软件包，要安装软件包，可选择present（安装）、installed（安装）、 latest（安装最新版本），删除软件包可选择absent、removed。

#### yum.yml
```
- hosts: 172.20.125.181  
  remote_user: root
  tasks:
  - name: install apache
    yum: name=httpd state=latest 
  - name: remove  nginx
    yum: name=nginx state=removed 
```
# user模块与group模块

user模块是请求的是useradd, userdel,   usermod三个指令  
goup模块请求的是groupadd, groupdel,   groupmod 三个指令  

name：指定用户名  
group：指定用户的主组  
groups：指定附加组，如果指定为('groups=')表示删除所有组。  
shell：指定默认shell  
state：设置帐号状态，不指定为默认为present，表示创建，指定值为absent表示删除  
remove：当使用状态为state=absent时使用，类似于userdel --remove选项，不仅仅删除用户，还会删除用户对应的家目录

#### useradd.yml
```
- hosts: test
  remote_user: root
  tasks:
  - name: create group
    group: name=puhu 
  - name: create users
    user: name={{item}} groups=root,puhu  shell=/sbin/nologin
    with_items: 创建的用户名字依次列出
    - kk1
    - kk2
    - kk3
```
#### userdel.yml
```
- hosts: test
  remote_user: root
  tasks:
  - name: create group
    group: name=puhu 
  - name: create users
    user: name={{item}} state=absent remove=true
    不加remove=true 默认只删除用户，加了家目录也会删除
    with_items:
    - kk1
    - kk2
    - kk3
```
# setup模块获取Ansible facts信息
Ansible facts是远程主机上的系统信息，主要包含IP地址，操作系统版本，网络设备，mac地址，内存、磁盘、硬件等信息。这些信息对于需要根据远程主机的信息作为执行条件操作的场景非常有用. Ansible提供了一个setup模块来收集远程主机的系统信息，这些facts信息可以直接以变量的形式使用.  
playbooks中，经常会用到的一个参数gather_facts就与该模块相关。gather_facts默认值为yes，也就是说，在使用Ansible对远程主机执行任何一个playbook之前，总会先通过setup模块获取facts，并将信息暂存在内存中，直到该playbook执行结束。  
[root@ansible playbook]# ansible 172.16.213.231  -m setup
```
172.20.125.181 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.20.125.181"
        ], 
        "ansible_all_ipv6_addresses": [], 
        "ansible_apparmor": {
            "status": "disabled"
        }, 
        "ansible_architecture": "x86_64", 
        "ansible_bios_date": "04/01/2014", 
        "ansible_bios_version": "8c24b4c", 
        "ansible_cmdline": {
            "BOOT_IMAGE": "/boot/vmlinuz-3.10.0-862.14.4.el7.x86_64", 
            "console": "ttyS0,115200n8", 
            "crashkernel": "auto", 
            "idle": "halt", 
            "net.ifnames": "0", 
            "noibrs": true, 
            "ro": true, 
            "root": "UUID=b98386f1-e6a8-44e3-9ce1-a50e59d9a170"
        }, 
        ...
```
所有数据格式都是JSON格式，facts还支持查看指定信息，如下所示：  
[root@ansible playbook]# ansible 172.16.213.231  -m setup -a 'filter=ansible_all_ipv4_addresses'
```
172.20.125.181 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.20.125.181"
        ], 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
```
# register、set_fact、debug模块

#### register_debug.yml
```
- hosts: test
  remote_user: root
  tasks:
  - name: get hostname
    shell: hostname
    register: get_hostname
    注册变量，意思就是给上边hostname命令的执行结果起了个get_hostname的变量名字
  - debug: var=get_hostname.stdout 
  debug调试，相当于打印变量，json格式,var是固定写法
  - debug: 'msg="主机名：{{get_hostname.stdout}}"'
  debug调试，相当于打印变量，json格式，两种不用的写法，
  下边这个能输入自定义的字符，比如“主机名：”
```
注意观察执行结果
```

PLAY [test] ******************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [172.20.125.181]

TASK [get hostname] **********************************************************************************************************
changed: [172.20.125.181]

TASK [debug] *****************************************************************************************************************
ok: [172.20.125.181] => {
    "get_hostname.stdout": "iZ8vb10vvaorjiql15sbfjZ"
}

TASK [debug] *****************************************************************************************************************
ok: [172.20.125.181] => {
    "msg": "主机名：iZ8vb10vvaorjiql15sbfjZ"
}

PLAY RECAP *******************************************************************************************************************
172.20.125.181             : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
注意观察上文的playbook里有get_hostname.stdout，由来是这样的
```
- hosts: test
  remote_user: root
  tasks:
  - name: get hostname
    shell: hostname
    register: get_hostname
  - debug: var=get_hostname 注意这里
  - debug: 'msg="主机名：{{get_hostname.stdout}}"'
```
执行结果
```
[root@iZ8vbe901lz5iykherzbihZ yml]# ansible-playbook  registerr_debug.yml 

PLAY [test] ******************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [172.20.125.181]

TASK [get hostname] **********************************************************************************************************
changed: [172.20.125.181]

TASK [debug] *****************************************************************************************************************
ok: [172.20.125.181] => {
    "get_hostname": {
        "changed": true, 
        "cmd": "hostname", 
        "delta": "0:00:00.024585", 
        "end": "2020-11-17 22:45:28.690297", 
        "failed": false, 
        "rc": 0, 
        "start": "2020-11-17 22:45:28.665712", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "iZ8vb10vvaorjiql15sbfjZ",  注意这里
        意思就是取的这个值
        "stdout_lines": [
            "iZ8vb10vvaorjiql15sbfjZ"
        ]
    }
}

TASK [debug] *****************************************************************************************************************
ok: [172.20.125.181] => {
    "msg": "主机名：iZ8vb10vvaorjiql15sbfjZ"
}

PLAY RECAP *******************************************************************************************************************
172.20.125.181             : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
# delegate_to、connection、和local_action模块
在本机执行命令 仅仅是执行命令  
例1:
```
- hosts: 172.20.125.181 
  remote_user: root
  gather_facts: true
  tasks:
   - name: connection
     shell: echo "connection . {{inventory_hostname}} $(hostname) ." >> /tmp/local.log
     connection: local
   - name: delegate_to
     shell: echo "delegate_to . {{inventory_hostname}} $(hostname) ." >> /tmp/local.log
     delegate_to: localhost
   - name: local_action
     local_action: shell echo "local_action. {{inventory_hostname}} $(hostname)" >> /tmp/local.log
```
执行结果
{inventory_hostname}} 变量获取的是远程主机的IP 并不是本地的  
$(hostname)命令是在本机执行的，获取的是本机的主机名
```
[root@ansible]# cat /tmp/local.log 
connection . 172.20.125.181 iZ8vbe901lz5iykherzbihZ .
delegate_to . 172.20.125.181 iZ8vbe901lz5iykherzbihZ .
local_action. 172.20.125.181 iZ8vbe901lz5iykherzbihZ
```
例2:
```
- hosts: test 
  remote_user: root
  gather_facts: true
  tasks:
   - name: connection
     shell: echo "connection . $(ip addr |grep inet |grep brd | awk '{print $2}') $(hostname) ." >> /tmp/local.log
     connection: local
   - name: delegate_to
     shell: echo "delegate_to . $(ip addr |grep inet |grep brd | awk '{print $2}') $(hostname) ." >> /tmp/local.log
     delegate_to: localhost
   - name: local_action
     local_action: shell echo "local_action. $(ip addr |grep inet |grep brd | awk '{print $2}') $(hostname)" >> /tmp/local.log
```
执行结果
命令都是在本地执行
```
[root@ansible]# cat /tmp/local.log 
connection . 172.20.125.180/20 iZ8vbe901lz5iykherzbihZ .
delegate_to . 172.20.125.180/20 iZ8vbe901lz5iykherzbihZ .
local_action. 172.20.125.180/20 iZ8vbe901lz5iykherzbihZ
```
