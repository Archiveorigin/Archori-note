# Ansible 自动化运维

<aside>
💡 Ansible 是一个简单的 强大的 无代理的 自动化运维工具

</aside>

# Ansible 大纲

```bash
1. Ansible Install
2. Inventory
3. Ansible Configuration
4. Manage Ansible Control Host
	Method 1. Adhoc
	Method 2. Playbook

```

## RH294 Visual Info

```bash
username : kiosk
password : redhat

username : root
password : Asimov

Contain 7 KVM Virual Host

		1. Workstation (Ansible Control Host)
		2. Server a (Ansible managed host)
		3. Server b (Ansible managed host)
		4. Server c (Ansible managed host)
		5. Server d (Ansible managed host)
		6. Bastion (Provide The Network)
		7. Classroom (Provide The script repobase .etc)
		
管理这些虚拟机 需要用到红帽提供的 rht tools
< red hat training tools >

启动顺序 ：
先启动classroom 再启动剩下的所有虚拟机
刚开始的环境没有虚拟机 需要使用命令拉取虚拟机

RH294 ~ : **rht-vmctl fullreset classroom**
Are you sure you want to full reset classroom? (y/n) y
Powering off classroom.
Full resetting classroom.
Creating virtual machine disk overlay for rh294-classroom-vda.qcow2
Starting classroom.

rht-vmctl 就是 rht tools 的命令
rht-vmctl fullreset 就是完全重置某个虚拟机 (虚拟机的删除重建)

RH294 etc : **rht-vmctl fullreset all** # 重置除了classroom之外所有的虚拟机
Are you sure you want to full reset bastion workstation servera serverb serverc serverd? (y/n) y
Powering off bastion.
Powering off workstation.
Powering off servera.
Powering off serverb.
Powering off serverc.
Powering off serverd.
Full resetting bastion.

# 恢复所有虚拟机的快照
rht-vmctl reset all
# 关闭所有节点
rht-vmctl poweroff all
# 开启所有节点
rht-vmctl start all
# 连接节点
ssh 节点名称
```

## 如何操作RH294环境

```bash
# 后面的练习都是通过rh294的物理操作系统连接到workstation操作
# workstation是Ansible Control节点
# Server a-d 是Ansible Managed节点
# 几乎所有的虚拟机root密码都是redhat
# student 是特权用户 密码是student
# workstation 
```

## Install Ansible

```bash
yum -y install ansible
```

## Inventory 文件

```bash
# Inventory文件定义了被管理的主机
# Inventory文件分为静态Inventory和动态Inventory

# 静态Inventory文件其实就是txt文本记录了被管理主机 只要不修改这个静态Inventory文件 被管理的主机就不会发生变化
# 动态Inventory文件是能动态输出被管理主机 动态Inventory文件原理就是一个脚本 大部分都是python脚本
# 动态Inventory文件能连接到某个管理系统的节点信息数据库 并将节点信息以特定的格式输出 大部分是json格式输出
# 动态Inventory文件输出的主机是管理节点信息数据库的内容 也就是说节点数据库中节点的变化会改变inventory文件的数据

# 编写INI格式的静态Inventory文件

RH294 ~ $ touch inventory
RH294 ~ $ cat << END inventory 
> servera
> serverb
> serverc
> serverd
> END
# 可以使用[]来定义主机组 和yum配置文件类似 可以是域名 也可以是IP 一个主机可以在多个组中

# 在Ansible的Inventory有两个特殊的组
	[all] # Inventory文件中所有的主机
	[ungrouped] # 不属于任何主机组的被管理主机

# 嵌套组定义
[whole]
servera
serverb
serverc
serverd

[producting]
servera
serverc

[testing]
serverb
serverd

[servers:children] # 需要在嵌套组后面加上 :children
producting
testing
```

### Inventory文件定义主机范围

```bash
# 定义192.168.0.0 - 192.168.0.255

# inventory
192.168.0.[0:255]

# 定义servera-d
server[a:d]
```

### 查看Inventory中主机信息

`ansible -i 配置文件路径 主机组名 --list-hosts`

```bash
RH294 ~ $ ansible -i inventory whole --list-hosts
  hosts (4):
    servera
    serverb
    serverc
    serverd
    
# ansible 命令的 -i 参数 可以指定Inventory文件的路径 并查看相关参数
```

### 默认Inventory文件

`/etc/ansible/hosts`

```bash
# 在ansible不加 -i 参数的时候默认读取的就是 /etc/ansible/hosts 文件
student ansible $ ansible all --list-hosts
  hosts (2):
    servera.lab.example.com
    serverb.lab.example.com
student ansible $ cat /etc/ansible/hosts
servera.lab.example.com
serverb.lab.example.com
```

## Ansible的配置文件

<aside>
💡 ansible的配置文件不是全局的 任何用户都可以有自己的ansible配置文件

</aside>

<aside>
💡 ansible在安装的时候就有一个缺省的配置文件

</aside>

配置文件地址： `/etc/ansible/ansible.cfg`

<aside>
💡 在实际的环境中 默认的配置文件不被使用 用户可以在自己的目录下创建ansible配置文件

</aside>

<aside>
📌 ansible的配置文件有4种形式可以指定 四种指定的方式优先级是不同的 如果没有任何其他的ansible配置文件 默认就会使用 `/etc/ansible/ansible.cfg` 这个配置文件 **这个文件的优先级是最低的 没有任何配置文件的前提下才会使用这个配置文件**

</aside>

**按照优先级排序ansible配置文件**

—最低—

`/etc/ansible/ansible.cfg`

`/home/username/.ansible.cfg`

`当前目录下.ansible.cfg`

`export ANSIBLE_CONFIG = path`

—最高—

### Ansible配置文件基本参数

```bash
# /etc/ansible/ansible.cfg

[defaults]
[inventory]
[privilege_escalation]
[paramiko_connection]
[ssh_connection]
[persistent_connection]
[accelerate]
[selinux]
[colors]
[diff]

# ansible 配置文件以 sector 作为划分 每一个方括号代表一个sector
```

```bash
[defaults]

inventory = /etc/ansible/hosts 
# 该配置文件 默认使用inventory文件为/etc/ansible/hosts
remote_user = username 
# 该配置文件使用username用户进行ssh连接
ask_pass = false 
# 使用username用户去ssh连接的时候不提示输入密码

[privilege_escalation]
# 如果remote_user用的是root 就不需要配置这部分 这部分用于提权

become = true 
# 表示需要提权
become_method = sudo
# 表示提权方式是sudo
become_user = root
# 表示提权到root用户
become_ask_pass = false
# 表示进行sudo操作时不输入密码

# 不是任何用户作为remote_user配置了提权就可以提权 必须在被管理主机里面配置sudoers文件 让remote_user有提权的能力才可以

```

## 使用 Ansible 的 ad hoc 格式管理主机

**ad hoc 命令格式**

```bash
ansible "host-pattern 主机组" -m "module 模块" -a 'module arguments' -i "inventory"

-m 使用ansible的模块
-a 模块的参数
-i 使用的inventory文件
```

### **ping模块**

<aside>
💡 ansible的ping模块在ad hoc中使用可以查看被控主机与控制主机的连通性 一般不加参数

</aside>

```bash
student deploy-manage $ **ansible intranetweb -m ping**
servera.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

### ansible-doc

列出ansible所有模块`ansible-doc -l` 

列出模块的信息 `ansible-doc 模块`

### **user模块**

```bash
# 添加用户
student deploy-manage $ **ansible internetweb -m user -a 'name=arc uid=5000 state=present'**
serverb.lab.example.com | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 5000,
    "home": "/home/arc",
    "name": "arc",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 5000
}
# 删除用户
student deploy-manage $ ansible internetweb -m user -a 'name=arc state=absent'
serverb.lab.example.com | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "force": false,
    "name": "arc",
    "remove": false,
    "state": "absent"
}
```

### command 模块

<aside>
💡 command 模块实现了管理主机通过ansible直接在被管理主机上执行Linux命令

</aside>

```bash
# 如果不指定-m参数 那么默认使用command模块
# 例如 ansible -a 'id'
# 同上 ansible -m command -a 'id'

student deploy-manage $ ansible all -m command -a 'id'
BECOME password: 
serverb.lab.example.com | CHANGED | rc=0 >>
uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

servera.lab.example.com | CHANGED | rc=0 >>
uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

localhost | CHANGED | rc=0 >>
uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

student deploy-manage $ ansible -m command -a 'pwd' all
serverb.lab.example.com | CHANGED | rc=0 >>
/home/student

servera.lab.example.com | CHANGED | rc=0 >>
/home/student

localhost | CHANGED | rc=0 >>
/home/student
```

### yum 模块

```bash
# yum 基本参数
name="选择安装的包名"
state="选择安装的版本 一般为latest"

student deploy-manage $ ansible internetweb -m yum -a 'name=tmux state=latest'
serverb.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": [
        "Installed: tmux"
    ]
}
```

### copy 模块

```bash
# copy 基本参数
src="要拷贝的文件 一般在本地"
dest="目的文件 一般在远程"
owner="所有人"
group="所属组"
mode="权限设置"
backup="重复是否备份 yes 或 no"
content="把一段文字拷贝到dest指定的文件"

student deploy-manage $ ansible localhost -m copy -a 'src=/home/student/deploy-manage/inventory dest=/home/student/deploy-manage/inventory backup=yes'
localhost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "checksum": "552b7a45c901a4e9a4cd551c26deed24095adeb1",
    "dest": "/home/student/deploy-manage/inventory",
    "gid": 1000,
    "group": "student",
    "mode": "0664",
    "owner": "student",
    "path": "/home/student/deploy-manage/inventory",
    "secontext": "unconfined_u:object_r:user_home_t:s0",
    "size": 137,
    "state": "file",
    "uid": 1000
}
```

## 使用 Ansible 的 playbook 格式管理主机

<aside>
💡 与ad hoc不同的是 playbook 可以运行多个task

</aside>

```bash
# playbook 采用 YAML 格式 
# YAML 是一种格式不是一种语言
# 在 YAML 中只能用空格 不能用TAB

# playbook 中有多个 play 每个 play 又有多个 task 每个 task 又有多个 模块
```

### 编辑 运行一个 playbook

```yaml
--- # 开头
- name: A first play # play名
  hosts: all         # 目标主机群
  tasks:             # 执行的task
          - name: creat a new user  # task名
            user:                   # 使用user模块
                    name: arc          # 用户名
                    uid: 1234          # uid
                    state: present     # 状态
```

```bash
student deploy-manage $ **ansible-playbook creatuser.yml** 

PLAY [A first play] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]
ok: [serverb.lab.example.com]
ok: [servera.lab.example.com]

TASK [creat a new user] ********************************************************
changed: [serverb.lab.example.com]
changed: [servera.lab.example.com]
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
servera.lab.example.com    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb.lab.example.com    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 检查 playbook 语法

`ansible-playbook --syntax-check 文件`

```bash
student deploy-manage $ **ansible-playbook --syntax-check creatuser.yml** 

playbook: creatuser.yml
```

### 《假装运行》Executing a Dry Run

```bash
student deploy-manage $ ansible-playbook -C creatuser.yml 

PLAY [A first play] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]
ok: [serverb.lab.example.com]
ok: [servera.lab.example.com]

TASK [creat a new user] ********************************************************
ok: [servera.lab.example.com]
ok: [serverb.lab.example.com]
ok: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
servera.lab.example.com    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb.lab.example.com    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 多 task 的 playbook 示例

```yaml
---
- name: A play get httpd
  hosts: whole
  tasks:
          - name: Install httpd
            yum:
                    name: httpd
                    state: latest

          - name: use index.html as web server content of page
            copy:
                    src: /home/student/testing/index.html
                    dest: /var/www/html/index.html

          - name: user service module to set httpd service as a started state
            service:
                    name: httpd
                    state: started
                    enabled: true
```

### 多 play 的 playbook 示例

```yaml
---
# This is a simple playbook with two plays

- name: First play
  hosts: web.example.com
  tasks:
    - name: First task
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Second task
      ansible.builtin.service:
        name: httpd
        enabled: true

- name: Second play
  hosts: database.example.com
  tasks:
    - name: First task
      ansible.builtin.service:
        name: mariadb
        enabled: true
```

### 设置不同 play 拥有不同的执行参数

```bash
例如
---
- name: ...
  hosts: ...
  remote_user: ...
  ...
  
# 包含的参数
remote_user: 设置ssh的用户
become: 设置是否提权 yes/no
hosts: 设置执行的主机群
```

### 使用 ansible-doc 查找模块用法

```bash
ansible-doc -l # 显示所有模块
ansible-doc 模块名 # 查看该模块详细用法
```

### 模块状态

```bash
stableinterface # 稳定状态 用法在未来不会改变
preview         # 前瞻状态 用法在未来可能改变
deprecated      # 滞留状态 模块在未来可能删除
removed         # 删除状态 模块在未来已经删除

core            # 受支持状态 提供免费技术支持
curated         # 第三方支持 接受付费技术支持
community       # 社区支持
```

### 字符串的输出状态

```bash
 string
'string'
"string"
这三个表达方式都属于字符串类
```

**无换行符**

```yaml
---
- name: this is a string testing
	hosts: all
	tasks:
			- name: modul one
				debug:
						msg: hello i m archiveorigin.
```

输出

```yaml
hello i m archiveorigin
```

文本编辑换行符

```yaml
---
- name: this is a string testing
	hosts: all
	tasks:
			- name: modul one
				debug:
						msg: |
						hello 
						i m 
						archiveorigin.
```

输出

```yaml
hello\ni m\narchiveorigin.
```

输出换行符

```yaml
---
- name: this is a string testing
	hosts: all
	tasks:
			- name: modul one
				debug:
						msg: >
						hello 
						i m 
						archiveorigin.
```

输出

```yaml
hello
i m
archiveorigin.
```

### YAML Dictionaries 字典

```yaml
name: svcrole
svcservice: httpd
svcport: 80 
```

可以写成

```yaml
{name: svcrole, svcservice: httpd, svcport:80} 
```

### YAML List 列表

```yaml
hosts:
	- servera
	- serverb
	- serverc
```

可以写成

```yaml
hosts: [servera, serverb, serverc]
```

### 限制 playbook 的作用范围

**Inventory**

```bash
[whole]
servera
serverb
serverc
serverd
localhost
```

**playbook**

```yaml
---
- name: First play
  hosts: whole
  tasks:
          - name: return ping
            ping: 
```

**Output**

```bash
student testing $ ansible-playbook servera check-play.yml 
ERROR! the playbook: servera could not be found

# 此时想要playbook在servera上运行 必须添加--limit=参数

student testing $ **ansible-playbook --limit=servera check-play.yml** 

PLAY [First play] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [servera]

TASK [return ping] *************************************************************
ok: [servera]

PLAY RECAP *********************************************************************
servera                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Ansible Variables 变量

<aside>
💡 使用 Ansible变量 让 playbook 集群部署简单高效

</aside>

**变量命名规范**

```bash
# 不能使用空格 小数点
# 不能以数字开头
# 尽量不用特殊符号 如$
```

### Defining Variables 定义变量

```bash
# 变量范围

Global scope  在整个playbook都生效
Play scope    在该play中有效
Host scope    在某个主机组有效

# 变量优先级

同样的变量名在不同级别被定义
更小范围的变量拥有更高优先级
在Inventory中定义的变量会被playbook中变量覆盖
Global scope > Play scope > Host scope
```

**定义 play 变量**

```yaml
---
- name: First play
  hosts: all
  vars:
          name: Archiveorigin
  tasks:
          - name: return Arh
            debug:
                    msg: "{{ name }} is very good person . "
```

**传入 Global 变量**

```bash
student testing $ **ansible-playbook return.yml -e "name=ours"**
 [WARNING]: Found variable using reserved name: name

PLAY [One play] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [serverb]
ok: [servera]

TASK [return Arh] **************************************************************
ok: [servera] => {
    "msg": "ours is very good person . "
}
ok: [serverb] => {
    "msg": "ours is very good person . "
}

PLAY RECAP *********************************************************************
servera                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

# 由此可见 通过命令行传入的 Global 变量拥有比 Play 变量更高的优先级 会覆盖play中的同名变量
```

**定义 Host 变量**

```bash
# Inventory
[whole]
servera name=arc
serverb name=ttt

# 回显
student testing $ ansible-playbook return.yml 

PLAY [One play] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [servera]
ok: [serverb]

TASK [return Arh] **************************************************************
ok: [servera] => {
    "msg": "arc is very good person . "
}
ok: [serverb] => {
    "msg": "ttt is very good person . "
}

PLAY RECAP *********************************************************************
servera                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0            
```

**通过 Inventory 设置 主机组 Host 变量**

```bash
# Inventory
[whole]
servera
serverb

[whole:vars]
name=arc
```

**通过变量文件定义play变量**

```bash
# playbook
---
- name: One play
  hosts: all
  **vars_files:
          variables.yml**
  tasks:
          - name: return Arh
            debug:
                    msg: "{{ name }} is very good person . "
                    
# variables.yml
name: arc

# 回显
student testing $ ansible-playbook return.yml 
 [WARNING]: Found variable using reserved name: name

PLAY [One play] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [servera]
ok: [serverb]

TASK [return Arh] **************************************************************
ok: [servera] => {
    "msg": "arc is very good person . "
}
ok: [serverb] => {
    "msg": "arc is very good person . "
}

PLAY RECAP *********************************************************************
servera                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### debug 模块

```bash
# debug 基本参数
msg: 输出文本 在应用变量时需要 "" 引用
var: 输出变量 后直接加变量名称
varbosity: 设置调试级别 整数类型 整数表示-v级别 例如4表示只有-vvvv才会应用这个debug模块

# playbook
---
- name: One play
  hosts: all
  vars_files:
          variables.yml
  tasks:
          - name: return Arh
            debug:          
                    var: name
                    
# 回显
student testing $ ansible-playbook return.yml 
 [WARNING]: Found variable using reserved name: name

PLAY [One play] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [servera]
ok: [serverb]

TASK [return Arh] **************************************************************
ok: [servera] => {
    "name": "arc"
}
ok: [serverb] => {
    "name": "arc"
}

PLAY RECAP *********************************************************************
servera                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

### 针对主机及主机组定义变量的文件

```bash
# 在Inventory所在的目录下创建 host_vars 和 group_vars 目录
# 两个目录下分别存放于主机/主机组同名的文件
# 文件中写变量的定义
# 这样便完成了对于主机/主机组的变量定义

# 目录结构
drwxrwxr-x. 2 student student   6 8月   5 02:57 group_vars
drwxrwxr-x. 2 student student  36 8月   5 02:58 host_vars
-rw-rw-r--. 1 student student  23 8月   4 06:17 index.html
-rw-rw-r--. 1 student student  26 8月   5 01:59 inventory
-rw-rw-r--. 1 student student 560 8月   4 08:40 playbook.yml
-rw-rw-r--. 1 student student 252 8月   5 02:56 return.yml

# inventory
[whole]
servera 
serverb

# ansible.cfg
[defaults]
inventory = /home/student/testing/inventory
remote_user = student
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false

# 两个变量文件
-rw-rw-r--. 1 student student 31 8月   5 02:58 servera
-rw-rw-r--. 1 student student 31 8月   5 02:58 serverb
student host_vars $ cat servera serverb
name: Arc
user: root
ttt: /aaa
name: AAA
user: user
ttt: /jei

# 回显
student testing $ ansible-playbook return.yml 
 [WARNING]: Found variable using reserved name: name

PLAY [One play] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [serverb]
ok: [servera]

TASK [return Arh] **************************************************************
ok: [servera] => {
    "name": "arc"
}
ok: [serverb] => {
    "name": "arc"
}

TASK [debug] *******************************************************************
ok: [servera] => {
    "user": "root"
}
ok: [serverb] => {
    "user": "user"
}

TASK [debug] *******************************************************************
ok: [servera] => {
    "ttt": "/aaa"
}
ok: [serverb] => {
    "ttt": "/jei"
}

PLAY RECAP *********************************************************************
servera                    : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb                    : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 生产环境中 应该怎么做？

```bash
# ansible 的部署应该遵循 高内聚 低耦合 的基本原则

playbook   -> task
inventory  -> host and host group
vars       -> vars.file
host vars  -> host_vars/
group vars -> group_vars/
```

### Arrays as Variables 变量层级

```bash
# 变量可以这样被定义
users:
	bjones:
		first_name: Bob
		last_name: Jones
		home_dir: /users/bjones
	acook:
		first_name: Anne
		last_name: Cook
		home_dir: /users/acook
		
# 调用方式
user.bjones.first_name
# 更建议的调用方式 （不会和python字典冲突)
users['bjones']['first_name']

# 编写位置
在playbook定义vars_files文件
变量文件必须用yml格式书写 以.yml结尾
```

### Register Variables 记录task运行状态

```bash
# 在编写 task 时 可以使用与task同级的register注册变量
# 语法： register: 变量名

# 示例 playbook
---
- name: First play
  hosts:
          - servera
          - serverb
  vars_files: ./vars.yml
  tasks:
          - name: output debug information
            debug:
                    msg: hello i will start the install

          - name: Install the httpd
            yum:
                    name: httpd
                    state: present
            **register: Install_status**
            ignore_errors: yes

          - debug:
                  var: Install_status
                  
          - debug:
                  var: Install_status['rc']
                  
# 输出
student testing $ ansible-playbook install.yml 

PLAY [First play] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [serverb]
ok: [servera]

TASK [output debug information] ************************************************
ok: [servera] => {
    "msg": "hello i will start the install"
}
ok: [serverb] => {
    "msg": "hello i will start the install"
}

TASK [Install the httpd] *******************************************************
fatal: [servera]: FAILED! => {"changed": false, "failures": ["httpd 没有能够与之匹配的软件包"], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring
fatal: [serverb]: FAILED! => {"changed": false, "failures": ["httpd 没有能够与之匹配的软件包"], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring

TASK [debug] *******************************************************************
ok: [servera] => {
    "Install_status": {
        "changed": false,
        "failed": true,
        "failures": [
            "httpd 没有能够与之匹配的软件包"
        ],
        "msg": "Failed to install some of the specified packages",
        "rc": 1,
        "results": []
    }
}
ok: [serverb] => {
    "Install_status": {
        "changed": false,
        "failed": true,
        "failures": [
            "httpd 没有能够与之匹配的软件包"
        ],
        "msg": "Failed to install some of the specified packages",
        "rc": 1,
        "results": []
    }
}

TASK [debug] *******************************************************************
ok: [servera] => {
    "Install_status['rc']": "1"
}
ok: [serverb] => {
    "Install_status['rc']": "1"
}

PLAY RECAP *********************************************************************
servera                    : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
serverb                    : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1

# 需要注意的是
# 使用 register 注册变量输出为json格式
# 同样遵循 “矩阵变量” 命名格式 可以逐级调用
```

### Ansible vault 加密敏感数据

<aside>
💡 在 Ansible 处理数据时 可能涉及敏感数据 例如密码 私钥等 这些敏感数据都会以明文形式存放于 playbook inventory ansible.cfg中 任何人访问这些文件都有可能暴露敏感数据 于是 Ansible 提供了一种加密数据的解决方案 即 **Ansible vault**

</aside>

```bash
# 基本语法
ansible-vault 参数 其他

# 基本参数
create  创建加密文件
view    查看加密文件
edit    编辑加密文件
rekey   重新设置密码
decrypt 解密加密文件
encrypt 加密明文文件

# 使用非交互方式查看加密文件
创建一个文件 文件中写密码
使用
ansible--vault-password-file=文件路径 参数 加密文件
```

**调用加密的变量文件执行一个playbook**

```bash
# ansible.cfg
[defaults]
inventory = ./inventory
remote_user = student
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false

# inventory
[whole]
servera
serverb

# values.yml
user: arc
escape: false

# vault_test.yml
user: arc
escape: false
student newtest $ cat vault_test.yml 
---
- name: First play to test the vault
  hosts:
          - servera
          - serverb
  vars_files: ./values.yml
  tasks:
          - name: output the string
            debug:
                    msg: "{{ user }}"

          - name: output the string
            debug:
                    var: escape
                    
# 加密 values.yml
student newtest $ **ansible-vault encrypt values.yml** 
New Vault password: 
Confirm New Vault password: 
Encryption successful

# 使用加密的 values.yml 运行 valut_test.yml
student newtest $ **ansible-playbook --vault-id @prompt vault_test.yml** 
Vault password (default): 

PLAY [First play to test the vault] ********************************************

TASK [Gathering Facts] *********************************************************
```

## Managing FACTS  管理 facts 变量

<aside>
💡 facts 变量记录了被管理主机的系统信息

</aside>

### 使用 ad hoc 方式查看 facts 变量

```bash
# 使用 setup 模块手动生成 facts 变量

ansible servera -m setup
输出略 ......
```

### 使用 debug 模块分析 facts 变量

<aside>
💡 facts以json格式输出 是一种“变量矩阵“

</aside>

```bash
# 小练习 使用debug模块抓取主机facts变量中 主机名称 IP地址 架构信息

# playbook
---
- name: First play to get facts
  hosts:
          - servera
          - serverb
  tasks:
          - name: first task to get facts of arch
            debug:
                    var: ansible_architecture

          - name: second task to get facts of ip4 info
            debug:
                    var: ansible_default_ipv4['address']

          - name: third task to get facts of hostname info
            debug:
                    var: ansible_domain
                    
# 回显
PLAY [First play to get facts] *************************************************

TASK [Gathering Facts] *********************************************************
ok: [serverb]
ok: [servera]

TASK [first task to get facts of arch] *****************************************
ok: [servera] => {
    "ansible_architecture": "x86_64"
}
ok: [serverb] => {
    "ansible_architecture": "x86_64"
}

TASK [second task to get facts of ip4 info] ************************************
ok: [servera] => {
    "ansible_default_ipv4['address']": "172.25.250.10"
}
ok: [serverb] => {
    "ansible_default_ipv4['address']": "172.25.250.11"
}

TASK [third task to get facts of hostname info] ********************************
ok: [servera] => {
    "ansible_domain": "lab.example.com"
}
ok: [serverb] => {
    "ansible_domain": "lab.example.com"
}

PLAY RECAP *********************************************************************
servera                    : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb                    : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

### 禁用facts收集以节省性能

```bash
# 在play开头中添加 gather_facts: no

# playbook

---
- name: First play to get facts
  hosts:
          - servera
          - serverb
  **gather_facts: no**
  tasks:
          - name: first task to get facts of arch
            debug:
                    var: ansible_architecture

          - name: second task to get facts of ip4 info
            debug:
                    var: ansible_default_ipv4['address']

          - name: third task to get facts of hostname info
            debug:
                    var: ansible_domain
```

### 定义facts变量

默认路径 `/etc/ansible/facts.d`

```bash
# 所有的自定义变量文件在被管理主机上以 .fact 结尾

"ansible_local": {},
# 这是一个预留专门给用户自定义变量的facts

# servera:/etc/ansible/test.fact
[pkg]
web_pkg = httpd

# ansible servera -m setup
  "ansible_local": {
            "test": {
                "pkg": {
                    "web_pkg": "httpd"
```

### Ansible magic 变量

```bash
student newtest $ **ansible servera -m debug -a "var=hostvars['servera']"**
servera | SUCCESS => {
    "hostvars['servera']": {
        "ansible_check_mode": false,
        "ansible_diff_mode": false,
        "ansible_facts": {},
        "ansible_forks": 5,
        "ansible_inventory_sources": [
            "/home/student/newtest/inventory"
        ],
        "ansible_playbook_python": "/usr/libexec/platform-python",
        "ansible_verbosity": 0,
        "ansible_version": {
            "full": "2.8.0",
            "major": 2,
            "minor": 8,
            "revision": 0,
            "string": "2.8.0"
        },
        "group_names": [
            "ungrouped"
        ],
        "groups": {
            "all": [
                "servera",
                "serverb"
            ],
            "ungrouped": [
                "servera",
                "serverb"
            ]
        },
        "inventory_dir": "/home/student/newtest",
        "inventory_file": "/home/student/newtest/inventory",
        "inventory_hostname": "servera",
        "inventory_hostname_short": "servera",
        "omit": "__omit_place_holder__8201075b8740f48eb03cf22f82ad79d39bd33f45",
        "playbook_dir": "/home/student/newtest"
    }
}

# 其中用处最大的 是 inventory_hostname
# 这个变量可以检测该主机在 inventory 如何表达
```