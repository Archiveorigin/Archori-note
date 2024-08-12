# RHCE9-RH294

## 基础环境介绍

```bash
# This environment is red hat ' s environment
foundation  kiosk:redhat root:Asimov
workstation  root:redhat
servera  root:redhat
serverb  root:redhat
bastion  root:redhat
utility  root:redhat

lab start users-superuser

# 这台机器包含了三套环境 分别是 rh124 rh134 和 rh294 对应了三本教材

# 如何切换环境？
# 切换环境时 需要在 foundation 上做
rht-setcourse 环境名称

# 查看当前环境
cat /etc/rht

# 确保classroom处于RUNNING状态
rht-vmctl status classroom
# 不在RUNNING状态需要重置
rht-vmctl reset classroom

# 现在环境的位置
D:\虚拟化工作站\官方环境

PS1=" \W $ "
```

# Ansible-navigator

<aside>
💡 Ansible-navigator 是取代 ansible 的 更友好的 工具

</aside>

## Ansible-navigator 配置文件

```bash
# ansible.cfg

[defaults]
inventory = inventory存放位置
remote_user = ssh连接用户
ask_pass = 是否询问密码 true/false

[privilege_escalation]
become = 是否提权 true/false
become_method = 提权方式 sudo/...
become_user = 提权用户 root/...
become_ask_pass = 提权是否询问密码 true/false

# ansible-navigator.yml

---
ansible-navigator:
  execution-environment:
    image: utility.lab.example.com/ee-supported-rhel8:latest
    pull:
      policy: missing
  playbook-artifact:
    enable: false
```

## Ansible-navigator 基本用法

```bash
ansible-navigator 子参数 参数

ansible-navigator inventory -i inventory -m stdout --list # 列出指定inventory中所有主机组
ansible-navigator inventory -i inventory -m stdout --graph all
@all:
  |--@development:
  |  |--servera.lab.example.com
  |--@london:
  |  |--serverd.lab.example.com
  |--@production:
  |  |--serverc.lab.example.com
  |  |--serverd.lab.example.com
  |--@testing:
  |  |--serverb.lab.example.com
  |--@ungrouped:
  |--@us:
  |  |--@mountainview:
  |  |  |--serverc.lab.example.com
  |  |--@raleigh:
  |  |  |--servera.lab.example.com
  |  |  |--serverb.lab.example.com
  |--@webservers:
  |  |--servera.lab.example.com
  |  |--serverb.lab.example.com
  |  |--serverc.lab.example.com
  |  |--serverd.lab.example.com
  
  ansible-navigator config # 列出包含的所有配置文件
  ansible-navigator run -m stdout playbook.yml
```

## Ansible-navigator playbook

### 单play的playbook练习

```yaml
# .yml
---
- name: first play
  hosts: servera.lab.example.com
  tasks:
    - name: Newbie exists with UID 4000
      ansible.builtin.user: # 在新版本中要求加入内容集名 (完全合规内容集名称) FQCN
        name: newbie
        uid: 4000
        state: present
```

```yaml
# site.yml

---
- name: First play book
  hosts: web
  tasks:
    - name: ENsure httpd package is present
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Correct index.html is present
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html

    - name: Ensure httpd is started
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
        
# feedback
workstation playbook-basic $ ansible-navigator run -m stdout site.yml 

PLAY [First play book] *********************************************************

TASK [Gathering Facts] *********************************************************
ok: [serverd.lab.example.com]
ok: [serverc.lab.example.com]

TASK [ENsure httpd package is present] *****************************************
changed: [serverd.lab.example.com]
changed: [serverc.lab.example.com]

TASK [Correct index.html is present] *******************************************
changed: [serverd.lab.example.com]
changed: [serverc.lab.example.com]

TASK [Ensure httpd is started] *************************************************
changed: [serverd.lab.example.com]
changed: [serverc.lab.example.com]

PLAY RECAP *********************************************************************
serverc.lab.example.com    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverd.lab.example.com    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 多play的playbook练习

```yaml
---
- name: Enable intranet services
  hosts: servera.lab.example.com
  become: true
  tasks:
    - name: Latest version of httpd and firewalld installed
      ansible.builtin.dnf:
        name:
          - httpd
          - firewalld
        state: latest

    - name: Tese html page is installled
      ansible.builtin.copy:
        content: "Welcome to the example.com intranet!\n"
        dest: /var/www/html/index.html

    - name: firewall enabled and running
      ansible.builtin.service:
        name: firewalld
        enabled: true
        state: started

    - name: firewall permits access to httpd service
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes

    - name: web server enabled and running
      ansible.builtin.service:
        name: httpd
        enabled: true
        state: started

- name: Test intranet web server
  hosts: localhost
  become: false
  tasks:
    - name: Connect to intranet wob server
      ansible.builtin.uri:
        url: http://servera.lab.example.com
        return_content: yes
        status_code: 200
```

### Variables 变量

```yaml
# 与ansible大同小异
```

### Managing Secrets 敏感信息加密

```yaml
# 傻逼navigator没有 用法和ansible一样

# ansible-vault 在编辑文件的时候会使用环境变量指定的编辑器
export EDITOR=编辑器

ansible-vault create 加密文件名
ansible-vault create --vault-password-file=file 加密文件名
ansible-vault view 加密文件名
ansible-vault edit 加密文件名
ansible-vault decrypt 现有加密文件
ansible-vault encrypt 明文文件
ansible-vault rekey 加密文件名

# Ansible-navigator 运行加密文件

ansible-navigator run -m stdout --playbook-artifact-enable false site.yml --vault-id @prompt
ansible-navigator run -m stdout site.yml --vault-password-file=vault-pass.txt
```

## Ansible-navigator Facts

```yaml
和 ansible 一样
```

## task 控制

### 条件判断 when

```yaml
# when紧跟在task之后 例如
---
- name: when e
  hosts: all
  vars: 
    service: httpd
  tasks:
    - name: test when
      ansible.builtin.dnf:
        name: "{{ service }}"
        state: latest
      **when: service is defined
      
# when 不是模块变量 所以必须在task的顶层缩进 将其置于模块之外
# 根据约定 when 一般放在task结尾**

# when的状态
when: variable1 == variable2
when: variable1 > variable2
when: variable1 < variable2
when: variable1 <= variable2
when: variable1 >= variable2
when: variable1 != variable2
when: variable1 is defined
when: variable1 is not defined
when: **Bool**
when: not **Bool**
when: variable1 in variable_list

# when的逻辑
when: variable1 and variable2
when: variable1 or variable2
```

### 循环结构 loop

```yaml
# 循环紧跟在task之后 例如

- name: postfix and devecot are running
  ansible.builtin.service:
    name: "{{ item }}"
    state: started
  loop:
    - postfix
    - dovecot
    
# 变量列表形式

vars:
  mail_services:
    - postfix
    - dovecot
tasks:
  - name: postfix and dovecot are running
  ansible.builtin.service:
    name: "{{ item }}"
    state: started
  loop: "{{ mail_services }}"
  
# 变量字典形式

- name: users exist and ate in the correct groups
  user:
    name: "{{ item['name'] }}"
    state: present
    groups: "{{ item['groups'] }}"
  loop:
    - name: jane
      groups: wheel
    - name: joe
      groups: root
```

**循环判断练习**

```yaml
---
- name: MariaDB server is running
  hosts: database_dev
  vars:
    mariadb_packages:
      - mariadb-server
      - python3-PyMySQL
        
  tasks:
    - name: MariaDB packages are installed
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ mariadb_packages }}"
      when: ansible_facts['distribution'] == "RedHat"
          
    - name: Start MariaDB_service
      ansible.builtin.service:
        name: mariadb
        state: started
        enabled: true
```

### 更新task状态

**设置failed状态**

```yaml
# 将 task 设置为 failed 状态 当满足条件时
tasks:
  - name: Run user creation script
    ansible.builtin.shell:
      - /usr/local/bin/create-users.sh
    register: command_result # 将脚本返回结果注册进command_result变量
    failed_when: "'Password missing' in command_result.stdout"
    # 判断如果字符串存在于变量则把task设置为failed状态
    
# 更好的写法
tasks: 
  - name: Run user creation script
    ansible.builtin.shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: yes
    
  - name: report script failure
    ansible.builtin.fail:
      msg: "The password is missing in the output"
    when: "'Password missing' in command_result.stdout"  
```

**设置changed状态**

```yaml
# 将 task 设置为 changed 状态 当满足条件时
- name: Validate httpd configuration
  ansible.builtin.command: httpd -t
  changed_when: false
  register: httpd_config_status
  # 此task将永远不是changed状态
  
# changed可以配合handlers使用
tasks:
  - ansible.builtin.shell:
    cmd: /usr/local/bin/upgrade-database
  register: command_result
  changed_when: "'Success' in command_result.stdout"
  notify:
    - restart_database
    
handlers:
  - name: restart_database
    ansible.builtin.service:
      name: mariadb
      state: restarted
```

## Implementing Handlers

<aside>
💡 Handlers 目的解决 playbook 多次执行带来的可能反复重启服务导致的性能损失

</aside>

```yaml
**Handlers的本质是一个task
它通过在上一个task的notify触发
一旦触发 Handlers会立刻执行包含其中的与之触发名称对应的task
在大多数情况下 在所有task运行完之后才会运行handlers 因为notify可能会通知多次
只有task中包含notify而且状态为changed才算一次有效通知**
```

**练习**

```yaml
---
- name: Web application server is deployed
  hosts: webapp
  vars:
    packages:
      - nginx
      - php-fpm
    web_service: nginx
    app_service: php-fpm
    resources_dir: /home/student/control-handlers/files
    web_config_src: "{{ resources_dir }}/nginx.conf.standard"
    web_config_dst: /etc/nginx/nginx.conf
    app_config_src: "{{ resources_dir }}/php-fpm.conf.standard"
    app_config_dst: /etc/php-fpm.conf

  tasks:
    - name: "{{ packages }} is installed"
      ansible.builtin.dnf:
        name: "{{ packages }}"
        state: present

    - name: make sure the web service is running
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: started
        enabled: true

    - name: Make sure the application service is running
      ansible.builtin.service:
        name: "{{ app_service }}"
        state: started
        enabled: true

    - name: The {{ web_config_dst }} file has been deployed
      ansible.builtin.copy:
        src: "{{ web_config_src }}"
        dest: "{{ web_config_dst }}"
        force: true
      notify:
        - restart web service

    - name: The {{ app_config_dst }} file has been deployed
      ansible.builtin.copy:
        src: "{{ app_config_src }}"
        dest: "{{ app_config_dst }}"
        force: true
      notify:
        - restart app service

  handlers:
    - name: restart web service
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: restarted

    - name: restart app service
      ansible.builtin.service:
        name: "{{ app_service }}"
        state: restarted
```

## Managing Task Errors 错误处理

### 预期错误处理

```yaml
# 当task出现预期错误 仍执行可以添加
ignore_errors: yes

# 例如
- name: Latest version if pkg
  ansible.builtin.dnf:
    name: notapkg
    state: latest
  ignore_errors: yes
```

```yaml
# 当task出现预期错误 playbook终止 仍然执行handlers
force_handlers: yes

# 例如
---
- hosts: all
  force_handlers: yes
  *省略...*
  
  handlers:
    - name: ...
```

### Blocks 结构

<aside>
💡 在 playbook 中 可以使用block结构使错误处理更加方便

</aside>

```yaml
# Block 可以包含多个模块使其成为整体
# 进而可以被一个 when 结构控制
---
- name: Block example
  hosts: all
  tasks:
    - name: install
      block:
      - name: package needed by dnf
        ansible.builtin.dnf:
          name: python3-dnf-plugin-versionlock
          state: present
      - name: lock version of tzdata
        ansible.builtin.lineinfile:
          dest: /etc/yum/plug......
          ......
      when: ansible_distribution == "RedHat"
      
# block用于将一组任务组织在一起，这些任务将作为一个整体执行。如果其中任何一个任务失败，控制权将转移到rescue部分。
# rescue部分中的任务在block中的任何任务失败时执行。它用于处理错误或进行补救操作
# always部分中的任务无论block中的任务是否成功或rescue中的任务是否执行，都会执行。它通常用于清理操作或确保某些最终任务始终运行。

tasks:
  - name: Upgrade DB
    block:
      - name: upgrade the database
        ansible.builtin.shell:
          cmd: /usr/local/lib/upgrade-database
    rescue:
      - name: revert the database upgrade
        ansible.builtin.shell:
          cmd: /usr/local/lib/revert-database
    always:
      - name: always restart the database
        ansible.builtin.service:
          name: mariadb
          state: restarted
```

## Ansible-navigator 远端文件管理

### 模块介绍

```yaml
# ansible.builtin.
blockinfile  在文件中插入或修改一个文本块
copy         复制本地文件到远端 并且设置相关参数
fetch        复制远端文件到本地
file         创建 设置文件 权限
lineinfile   修改文件中特定行
stat         获取和检查文件的信息与存活状态
patch        应用或撤销补丁文件
synchronize  同步文件和目录
```

## Ansible-navigator Jinja2

```yaml
# 组成
Jinja2 模版中包含 数据 变量 表达式 (data variables expressions)

# 表达规范
{{ EXPR }}    变量
{% EXPR %}    表达式
{# COMMENT #} 注释

# ansible.cfg
ansible_managed = Ansible managed
目的用于j2模版中声明 {{ ansible_managed }}

# 循环
{% for i in users %} # users为值列表
{{ i }} #此行直接作为输出
{% endfor %}}

# 循环 条件判断
{% for i in users if not i == "root" %} # 如果i不是root才将值赋给i
User number {{ loop.index }} - {{ i }}  # 此行作为输出 其中loop.index为循环索引 输出为值在循环列表中的位数
{% endfor %}

# 循环 变量矩阵
{% for i in groups['myhosts'] %}
{{ i }}
{% endfor %}

# 循环 示例
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }}
{{ hostvars[host]['ansible_facts']['fqdn'] }}
{{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %} 
所有主机作为host传入
获取 ipv4地址 完全合规域名 主机名

# 条件
{% if Bool %}
{{ result }}
{% endif %}

# 注意事项
管理Jinja2模版需要用到
ansible.builtin.template

# 示例
---
- name: Configure SOE
  hosts: all
  remote_user: devops
  become: true
  vars:
    - system_owner: clyde@example.com
  tasks:
    - name: Configure /etc/motd
      **ansible.builtin.template:**
        src: motd.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: 0644
        
# 登陆提示文件
/etc/motd.d/cockpit/
/etc/motd
```

## Managing Complex Playbooks

### 通配符

```yaml
# playbook.yml
---
- name: 
  hosts: '*'
  
# 通配符
'*' 匹配所有主机
'!hostname,hostgroup' 匹配不属于hostname和hostgroup中的主机
'192.168.2.*' 匹配2网段
'lab,&datacentoer' 取交集
```

### 包含 和 导入 文件

```yaml
# 包含 和 导入文件有两种方式
include
import

include 是playbook运行到加载 dynamic 动态加载
import  是playbook运行前加载 static  静态加载

被包含/导入的文件如果只包含task 要加载在task层面
被包含/导入的文件如果包含playbook 要加载在play层面
```

**示例**

```yaml
# 环境
./
├── ansible.cfg
├── inventory
├── plays/
│   └── test.yml
└── tasks/
    ├── environment.yml
    ├── firewall.yml
    └── placeholder.yml
    
# test.yml
---
- name: Test web service
  hosts: workstation
  become: false
  tasks:
    - name: Connect to internet web server
      ansible.builtin.uri:
        url: "{{ url }}"
        status_code: 200
        
# environment.yml
---
  - name: Install the {{ package }} package
    ansible.builtin.dnf:
      name: "{{ package }}"
      state: latest
  - name: Start the {{ service }} service
    ansible.builtin.service:
      name: "{{ service }}"
      enabled: true
      state: started
      
# firewall.yml
---
  - name: Install the firewall
    ansible.builtin.dnf:
      name: "{{ firewall_pkg }}"
      state: latest

  - name: Start the firewall
    ansible.builtin.service:
      state: started
      name: "{{ firewall_svc }}"
      enabled: true

  - name: Open the port for {{ rule }}
    ansible.posix.firewalld:
      service: "{{ item }}"
      immediate: true
      permanent: true
      state: enabled
    loop: "{{ rule }}"
    
# placeholder.yml
---
  - name: Create placeholder file
    ansible.builtin.copy:
      content: "{{ ansible_facts['fqdn'] }} has been customized using Ansible.\n"
      dest: "{{ file }}"
      
# playbook.yml
---
- name: Configure web server
  hosts: servera.lab.example.com
  tasks:
    - name: test include
      include_tasks: tasks/environment.yml
      vars:
        package: httpd
        service: httpd
          
    - name: import
      import_tasks: tasks/firewall.yml
      vars:
        firewall_pkg: firewalld
        firewall_svc: firewalld
        rule:
          - http
          - https
            
    - name: Import the place holder task file and set the variable
      import_tasks: tasks/placeholder.yml
      vars:
        file: /var/www/html/index.html
          
- name: improt playbook test
  import_playbook: plays/test.yml
  vars: 
    url: 'http://servera.lab.example.com'
```

## Ansible-navigator Roles & Collections

### Ansible-navigator Roles

<aside>
💡 Roles 就是以一种标准化格式将 Ansible 中的 变量 文件 模版 和其他东西打包的工具 方便重用代码

</aside>

```yaml
# 创建roles
# 定义roles目录结构
# 使用roles
```

创建roles

```yaml
ansible-galaxy init roles名称

workstation roles $ ansible-galaxy init myvhost
- Role myvhost was created successfully
workstation roles $ ls
myvhost
workstation roles $ tree myvhost/
myvhost/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
    
# tasks执行优先级
pre_tasks > tasks > roles > post_tasks
```

调用roles

```yaml
---
- name: Use myvhost role playbook
  hosts: webservers
  pre_tasks:
    - name: pre_tasks message
      ansible.builtin.debug:
        msg: 'Ensure web server configuration.'

  roles:
    - myvhost

  post_tasks:
    - name: HTML content is installed
      ansible.builtin.copy:
        src: roles/myvhost/files/html/
        dest: "/var/www/vhosts/{{ ansible_hostname }}"

    - name: post_tasks message
      ansible.builtin.debug:
        msg: 'Web server is configured.'
```

**从外部源部署roles**

```yaml
# 通过Ansible Galaxy
ansible-galaxy install role_name

# 使用requirements.yml 文件
- src: geerlingguy.apache
  version: 3.0.0
  
- src: git+https://github.com/username/repo.git
  version: master
  
ansible-galaxy install -r requirements.yml -p 指定安装目录
```

### Ansible-navigator Collections

<aside>
💡 Ansible-navigator Collections 用于扩展ansible的内容

</aside>

**查看当前 collections**

```yaml
# **ansible-galaxy collection list**

workstation role-collections $ **ansible-galaxy collection list**

# /usr/share/ansible/collections/ansible_collections
Collection               Version
------------------------ -------
redhat.rhel_system_roles 1.16.2 
```

安装collection

```yaml
ansible-galaxy collection install <collection> -p <dir>
```

## Ansible Troubleshooting

```yaml
# 和 Ansible 一样
```

## Ansible 实现 Linux 管理自动化

### Ansible 实现 软件包管理

```yaml
ansible.builtin.dnf:
•	name：要安装或删除的软件包名称。可以是一个列表。
•	state：指定软件包的状态，可以是 present，absent，latest。
•	enablerepo：要启用的特定软件仓库。
•	disablerepo：要禁用的特定软件仓库。
•	update_cache：布尔值，是否在安装前更新Yum缓存。默认 no。
```

### Ansible 实现 用户管理和认证介绍

```yaml
ansible.builtin.user:
•	name：要管理的用户名。
•	state：present 表示创建或更新用户，absent 表示删除用户。
•	password：用户的加密密码。可以使用 mkpasswd 命令生成。
•	groups：用户应加入的组列表。
•	shell：用户的登录 shell。
•	home：用户的 home 目录路径。

ansible.builtin.group:
•	name：要管理的组名。
•	state：present 表示创建组，absent 表示删除组。
•	gid：指定组ID（GID）。

ansible.builtin.authorized_key:
authorized_key 模块用于管理远程主机上的用户SSH公钥，以实现无密码登录。
•	基本参数：
•	user：要管理公钥的用户。
•	state：present 表示添加公钥，absent 表示移除公钥。
•	key：公钥内容，可以从文件中读取。
```

### Ansible 实现 管理启动进程调度进程

```yaml
ansiblt.posix.at:
• command 执行一次性计划任务命令
• count 计数 单位为units
• units 计数单位
• unique 如果存在则不添加 yes/no

ansible.builtin.cron:
• name 描述
• user 用户的cron
• job 命令
• minute 分
• hour 时
• weekday 星期
• day 日
• month 月
• state 状态 present/absent

ansible.builtin.systemd:
• name 服务名称
• state 状态 stopped/started
• enabled 自启动状态 yes/no
```

### Ansible 实现存储管理

Ansible 可以通过多种模块来管理 Linux 系统中的存储设备和文件系统。以下是一些常用的模块和示例：

### 管理磁盘分区

使用 `ansible.builtin.parted` 模块可以管理磁盘分区。

```yaml
- name: 创建新分区
  ansible.builtin.parted:
    device: /dev/sdb
    number: 1
    state: present
    part_start: 1
    part_end: 100%
    fs_type: ext4

```

### 创建文件系统

使用 `ansible.builtin.filesystem` 模块来创建文件系统。

```yaml
- name: 创建 ext4 文件系统
  ansible.builtin.filesystem:
    fstype: ext4
    dev: /dev/sdb1

```

### 挂载文件系统

使用 `ansible.builtin.mount` 模块来挂载文件系统。

```yaml
- name: 挂载文件系统
  ansible.builtin.mount:
    path: /mnt/data
    src: /dev/sdb1
    fstype: ext4
    state: mounted

```

### 管理 LVM 逻辑卷

使用 `ansible.builtin.lvol` 模块来管理 LVM 逻辑卷。

```yaml
- name: 创建物理卷
  ansible.builtin.command: pvcreate /dev/sdc

- name: 创建卷组
  ansible.builtin.command: vgcreate myvg /dev/sdc

- name: 创建逻辑卷
  ansible.builtin.lvol:
    vg: myvg
    lv: mylv
    size: 5G

- name: 创建文件系统
  ansible.builtin.filesystem:
    fstype: ext4
    dev: /dev/myvg/mylv

- name: 挂载逻辑卷
  ansible.builtin.mount:
    path: /mnt/lvdata
    src: /dev/myvg/mylv
    fstype: ext4
    state: mounted

```

### Ansible 实现网络管理

Ansible 可以通过多个模块来管理网络设备和配置。以下是一些常用的网络管理模块及其示例：

### 管理网络接口

使用 `ansible.builtin.network` 模块可以管理网络接口的配置。

```yaml
- name: 配置网络接口
  ansible.builtin.network:
    name: eth0
    state: up
    ip: 192.168.1.100
    netmask: 255.255.255.0
    gateway: 192.168.1.1

```

### 管理防火墙规则

使用 `ansible.posix.firewalld` 模块来管理防火墙规则。

```yaml
- name: 打开 HTTP 服务端口
  ansible.posix.firewalld:
    service: http
    permanent: true
    state: enabled

- name: 打开自定义端口
  ansible.posix.firewalld:
    port: 8080/tcp
    permanent: true
    state: enabled

```

### 配置主机名和 DNS

使用 `ansible.builtin.hostname` 和 `ansible.builtin.lineinfile` 模块来配置主机名和 DNS。

```yaml
- name: 设置主机名
  ansible.builtin.hostname:
    name: server.example.com

- name: 配置 DNS 服务器
  ansible.builtin.lineinfile:
    path: /etc/resolv.conf
    regexp: '^nameserver'
    line: 'nameserver 8.8.8.8'
    state: present

```

### 配置静态路由

使用 `ansible.builtin.command` 模块来配置静态路由。

```yaml
- name: 添加静态路由
  ansible.builtin.command: ip route add 10.0.0.0/8 via 192.168.1.1

- name: 确保静态路由持久化
  ansible.builtin.lineinfile:
    path: /etc/sysconfig/network-scripts/route-eth0
    line: '10.0.0.0/8 via 192.168.1.1'
    state: present

```

### 配置网络时间协议（NTP）

使用 `ansible.builtin.yum` 和 `ansible.builtin.template` 模块来配置 NTP。

```yaml
- name: 安装 NTP 服务
  ansible.builtin.yum:
    name: ntp
    state: present

- name: 配置 NTP
  ansible.builtin.template:
    src: templates/ntp.conf.j2
    dest: /etc/ntp.conf
    owner: root
    group: root
    mode: 0644

- name: 启动并启用 NTP 服务
  ansible.builtin.service:
    name: ntpd
    state: started
    enabled: true
```