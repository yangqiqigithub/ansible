# palybook简介
playbook字面意思，即剧本，现实中由演员按照剧本表演，在Ansible中，这次由计算机进行表演，由计算机安装，部署应用，提供对外服务，以及组织计算机处理各种各样的事情。
执行一些简单的任务，使用命令行模式可以方便的解决问题，但是有时一个设施过于复杂，需要大量的操作时候，执行命令行模式是不适合的，这时最好使用playbook，就像执行shell命令与写shell脚本一样，也可以理解为批处理任务，不过playbook有自己的语法格式。

# playbook文件格式
playbook文件由YAML语言编写。 YAML是一个类似 XML、JSON的标记性语言，YAML强调以数据为中心，并不是以标识语言为重点。因而YAML本身的定义比较简单，号称“一种人性化的数据格式语言”。首先学习了解一下YAML的格式，对后面书写playbook很有帮助。以下为playbook常用到的YAML格式。
```
大小写敏感
使用空格作为嵌套缩进工具，缩进时不允许使用Tab键
缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
使用“-”（横线） + 单个空格：表示单个列表项
使用 “:”（冒号） + 空格：表示单个键值对
使用"{}"表示一个键值表
```
playbook文件是通过ansible-playbook命令进行解析的，ansbile-playbook命令会根据自上而下的顺序依次执行playbook文件中的内容。同时，playbook开创了很多特性,它可以允许传输某个命令的状态到后面的指令,它也可以从一台机器的文件中抓取内容并附为变量，然后在另一台机器中使用，这使得playbook可以实现一些复杂的部署机制,这是ansible命令无法实现的。

# palybook的构成
playbook是由一个或多个“play”组成的列表。play的主要功能在于，将事先合并为一组的主机装扮成事先通过ansible定义好的角色。将多个play组织在一个playbook中就可以让它们联同起来按事先编排的机制完成一系列复杂的任务。

其主要有以下四部分构成  
  target部分：   定义将要执行 playbook 的远程主机组  
  variable部分： 定义playbook运行时需要使用的变量  
  task部分：     定义将要在远程主机上执行的任务列表  
  handler部分：  定义task 执行完成以后需要调用的任务  

每部分构成的介绍如下：  
1）、Hosts和Users

playbook中的每一个play的目的都是为了让某个或某些主机以某个指定的用户身份执行任务。

hosts：用于指定要执行指定任务的主机，每个playbook都必须指定hosts，hosts也可以使用通配符格式。主机或主机组在inventory清单中指定，可以使用系统默认的/etc/ansible/hosts，也可以自己编辑，在运行的时候加上-i选项，可指定自定义主机清单的位置。在运行清单文件的时候，--list-hosts选项会显示那些主机将会参与执行任务的过程中。

remote_user：用于指定在远程主机上执行任务的用户。可以指定任意用户，也可以使用sudo，但是用户必须要有执行相应任务的权限。

（2）、任务列表（tasks list）  
play的主体部分是task list。

task list中的各任务按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个任务后再开始第二个。在运行自上而下某playbook时，如果中途发生错误，则所有已执行任务都将回滚，因此在更正playbook后需要重新执行一次。 

task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的（幂等性； 即一个命令,即使执行一次或多次, 其结果也一样），这意味着多次执行是安全的，因为其结果均一致。tasks包含name和要执行的模块，name是可选的，只是为了便于用户阅读，建议加上去，模块是必须的，同时也要给予模块相应的参数。

定义tasks推荐使用module: options”的格式，例如：
service: name=httpd state=running

（4）、tags

tags用于让用户选择运行或略过playbook中的部分代码。ansible具有幂等性，因此会自动跳过没有变化的部分；但是当一个playbook任务比较多时，一个一个的判断每个部分是否发生了变化，也需要很长时间。因此，如果确定某些部分没有发生变化，就可以通过tags跳过这些代码片断。

（5）、handlers 

用于当关注的资源发生变化时采取一定的操作。handlers是和“notify”配合使用的。

“notify”这个动作可用于在每个play的最后被触发，这样可以避免多次有改变发生时，每次都执行指定的操作，通过“notify”，仅在所有的变化发生完成后一次性地执行指定操作。
在notify中列出的操作称为handler，也就是说notify用来调用handler中定义的操作。 

注意：在 notify中定义的内容一定要和handlers中定义的“ - name”内容一样，这样才能达到触发的效果，否则会不生效。

# Playbook执行结果解析

使用ansible-playbook运行playbook文件，输出的内容为JSON格式。并且由不同颜色组成，便于识别。一般而言，输出内容中：  
绿色代表执行成功，但系统保持原样  
黄色代表系统状态发生改变，也就是执行的操作生效  
红色代表执行失败，会显示错误信息。  


# playbook举例
#### shell、command、raw 模块
命令模块

command 执行简单的命令，不支持"<"，">"，"|"，";"，"&"等符号  

shell 基本什么命令都能执行，支持"<"，">"，"|"，";"，"&"等符号,执行命令的时候使用的是/bin/sh，所以shell模块可以执行任何命令

raw 功能类似与前面说的command、shell能够完成的操作，raw也都能完成。不同的是，raw模块不需要远程主机上的python环境。

#### 选项
chdir 在执行命令前，进入到指定目录中  
creates 判断指定文件是否存在，如果存在，不执行后面的操作   
removes 判断指定文件是否存在，如果存在，执行后面的操作

#### 举例shell.yml
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
