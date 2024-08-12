# 防火墙 与 SELinux

## Linux中的防火墙介绍

<aside>
⚠️ Linux本身不具备防火墙功能 所有流量均由Linux内核管控 Linux中的防火墙功能是通过内核中 `net_filter` 模块提供 只要内核有这个模块 那么Linux就具备防火墙功能

</aside>

## iptables介绍和基础操作

<aside>
💡 iptables就是一个客户端 该客户端直接和内核空间net_filter交互 告诉net_filter什么样的流量需要放行 什么样的流量需要阻塞

</aside>

Linux的防火墙功能是由net_filter提供的

iptables是一个客户端 客户端用来接受用户命令

iptables将用户的命令转换成对应的net_fliter里面的过滤规则

**查看iptables的表**

`iptables -L`

```bash
[root@CitProbe ~] **iptables -L**
Chain INPUT (policy ACCEPT) #所有入站流量 默认都被允许
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)#所有转发流量 默认都被允许
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)#所有的出站流量 默认都被允许
target     prot opt source               destination
```

**查看PREROUTING和POSTROUTING规则**

`iptables -L -t nat`

```bash
[root@CitProbe ~] **iptables -L -t nat**
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
```

**什么是PREROUTING?**

<aside>
💡 **PREROUTING** 链是数据包进入系统时 第一个处理的链 主要用于 **DNAT (目的地址转换)** 修改数据包的目的地址 实现转发数据至其他服务器

</aside>

**什么是POSTROUTING?**

<aside>
💡 **POSTROUTING** 链是数据包路由决定之后 最后一个处理的链 主要用于 **SNAT(源地址转换)** 修改数据包的源地址 使其看起来是从不同的地址发送的

</aside>

**iptables主要参数**

```bash
iptables -F  # 清空所有的防火墙规则
iptables -X  # 删除用户自定义的空链
iptables -Z  # 清空计数
```

```bash
# iptable 
-t, --table table 对指定的表 table 进行操作， table 必须是 raw， nat，filter，mangle 中的一个。如果不指定此选项，默认的是 filter 表。

# 通用匹配：源地址目标地址的匹配
-p：指定要匹配的数据包协议类型；
-s, --source [!] address[/mask] ：把指定的一个／一组地址作为源地址，按此规则进行过滤。当后面没有 mask 时，address 是一个地址，比如：192.168.1.1；当 mask 指定时，可以表示一组范围内的地址，比如：192.168.1.0/255.255.255.0。
-d, --destination [!] address[/mask] ：地址格式同上，但这里是指定地址为目的地址，按此进行过滤。
-i, --in-interface [!] <网络接口name> ：指定数据包的来自来自网络接口，比如最常见的 eth0 。注意：它只对 INPUT，FORWARD，PREROUTING 这三个链起作用。如果没有指定此选项， 说明可以来自任何一个网络接口。同前面类似，"!" 表示取反。
-o, --out-interface [!] <网络接口name> ：指定数据包出去的网络接口。只对 OUTPUT，FORWARD，POSTROUTING 三个链起作用。

# 查看管理命令
-L, --list [chain] 列出链 chain 上面的所有规则，如果没有指定链，列出表上所有链的所有规则。

# 规则管理命令
-A, --append chain rule-specification 在指定链 chain 的末尾插入指定的规则，也就是说，这条规则会被放到最后，最后才会被执行。规则是由后面的匹配来指定。
-I, --insert chain [rulenum] rule-specification 在链 chain 中的指定位置插入一条或多条规则。如果指定的规则号是1，则在链的头部插入。这也是默认的情况，如果没有指定规则号。
-D, --delete chain rule-specification -D, --delete chain rulenum 在指定的链 chain 中删除一个或多个指定规则。
-R num：Replays替换/修改第几条规则

# 链管理命令（这都是立即生效的）
-P, --policy chain target ：为指定的链 chain 设置策略 target。注意，只有内置的链才允许有策略，用户自定义的是不允许的。
-F, --flush [chain] 清空指定链 chain 上面的所有规则。如果没有指定链，清空该表上所有链的所有规则。
-N, --new-chain chain 用指定的名字创建一个新的链。
-X, --delete-chain [chain] ：删除指定的链，这个链必须没有被其它任何规则引用，而且这条上必须没有任何规则。如果没有指定链名，则会删除该表中所有非内置的链。
-E, --rename-chain old-chain new-chain ：用指定的新名字去重命名指定的链。这并不会对链内部造成任何影响。
-Z, --zero [chain] ：把指定链，或者表中的所有链上的所有计数器清零。

-j, --jump target <指定目标> ：即满足某条件时该执行什么样的动作。target 可以是内置的目标，比如 ACCEPT，也可以是用户自定义的链。
-h：显示帮助信息；

```

<aside>
💡 源IP 目的IP 源端口 目的端口 协议 称为 **五源组**

</aside>

<aside>
🚧 Task  ：了解NAT 并 熟练使用iptables

</aside>

## firewalld原理

<aside>
💡 firewalld是一个服务 这个服务提供了防火墙配置的工具 firewalld提供了一个更加简单的方式来配置防火墙 firewalld的职责就是将指令转为iptables的规则 并且支持补全操作

</aside>

firewalld提供了zone的概念 firewalld将系统划分了一个个zone zone的边界取决于网卡 一个网卡只能属于一个zone

firewalld的zone里面有网卡 有规则（rule）

如果一个网卡属于一个zone 应用于该zone的规则会应用于zone内所有的网卡

## firewalld操作

### 查看所有zone

`firewall-cmd --list-all-zone`

### 列出缺省zone规则

**默认缺省zone为public**

`firewall-cmd --list-all`

```bash
[root@CitProbe ~] **firewall-cmd --list-all**
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160 ens224 ens256 ens256-vlan10
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

<aside>
💡 firewall-cmd添加规则都需要指定zone 但重复指定zone太麻烦 所以firewall指定了public为缺省zone

</aside>

**查看缺省zone**

`firewall-cmd --get-default-zone`

```bash
[root@CitProbe ~] **firewall-cmd --get-default-zone**
public
```

**修改缺省zone**

`firewall-cmd --set-default-zone=zone名`

```bash
[root@CitProbe ~] **firewall-cmd --set-default-zone=home**
success
[root@CitProbe ~] firewall-cmd --get-default-zone
home
```

<aside>
💡 默认Linux所有网卡都在默认zone里

</aside>

<aside>
⚠️ 像这样的防火墙 无论firewalld或者iptables 都管理的是内核管理的网卡 如果网卡不受内核管理 则防火墙规则对这种网卡无效

</aside>

**列出zone中所有网卡**

`firewall-cmd --list-interface --zone=zone名`

```bash
[root@CitProbe ~] **firewall-cmd --list-interfaces**
ens224 ens160
```

### service规则

**放行指定服务的端口和流量**

```bash
[root@CitProbe yum.repos.d] **firewall-cmd --add-service=http**
success
[root@CitProbe yum.repos.d] **firewall-cmd --list-services**
cockpit dhcpv6-client http ssh
```

<aside>
⚠️ service规则的弊端 ：service规则的确可以省略很多步骤 但是service规则放行的端口和流量都在 `usr/lib/firewalld/services` 中 以xml格式记录每个服务的常用端口号 所以放行服务相当于放行文件中记录的端口号

</aside>

### port规则

**放行指定端口的流量**

`firewall-cmd --add-port=端口号/协议`

```bash
[root@CitProbe services] firewall-cmd --add-port=80/tcp
success
[root@CitProbe services] firewall-cmd --add-port=80/udp
success
[root@CitProbe services] firewall-cmd --list-port
80/tcp 80/udp
```

### rich rule 规则 — 针对五源组的流量控制策略

**类似于iptables语法**

`firewall-cmd --add-rich-rule="rule family=ipv4 source address=<address> port port=<port number> protocol=<protocol name> <action>`

```bash
[root@CitProbe services] **firewall-cmd --add-rich-rule="rule family=ipv4 source address=192.168.80.101 port port=22 protocol=tcp reject"**
success
[root@CitProbe services] firewall-cmd --list-rich-rules
rule family="ipv4" source address="192.168.80.101" port port="22" protocol="tcp" reject
```

### 将规则保存

**在规则后面加上 `--permanent` 参数**

**或者运行`firewall-cmd --runtime-to-permanent`**

## SELinux (Security Enhanced Linux)

**DAC 与 MAC**

```bash
**DAC** **(Discretionary Access Control)** 即 自主访问控制
进程所拥有的权限与执行它的用户的权限相同

**MAC** **(Mandatory Access Control)** 即 ****强制访问控制
进程在SELinux系统中做任何事情 都必须先在安全策略配置文件中赋予权限
凡是没有出现在安全策略配置文件中的权限 进程就没有权限
```

### SE Linux 安全上下文

**SELinux如何保证访问安全？**

<aside>
💡 SELinux 使用 MAC 强制访问控制方法 即给每个进程 文件 都标记一个标签 进程只能访问对自己有权限的文件 其他操作会被拒绝

</aside>

**查看文件的标签**

```bash
[root@CitProbe ~] ls -lZ
total 16
-rw-rwx---+ 1 root root system_u:object_r:admin_home_t:s0      950 Jun 15 08:10 anaconda-ks.cfg
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jun 15 09:28 Desktop
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jun 15 09:28 Documents
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jun 15 09:28 Downloads
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jun 15 09:28 Music
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0   53 Jul  8 13:14 Pictures
-rw-r--r--. 1 root root unconfined_u:object_r:admin_home_t:s0 5411 Jul  7 10:19 pstree
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jun 15 09:28 Public
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jun 15 09:28 Templates
drwxr-xr-x+ 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jul  4 15:06 test
drwxrwsrwx. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jul  4 12:43 ts
-rw-r--r--. 1 root root unconfined_u:object_r:admin_home_t:s0   11 Jul  4 09:49 ttt
drwxr-xr-x. 2 root root unconfined_u:object_r:admin_home_t:s0    6 Jun 15 09:28 Videos
```

**查找进程的标签**

```bash
[root@CitProbe ~] ps auxZ | grep http
system_u:system_r:httpd_t:s0    root        6613  0.0  0.3  20332 11648 ?        Ss   14:34   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6615  0.0  0.1  21668  7356 ?        S    14:34   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6616  0.0  0.4 1669368 15232 ?       Sl   14:34   0:02 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6617  0.0  0.3 1538232 13872 ?       Sl   14:34   0:02 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6618  0.0  0.4 1538232 15228 ?       Sl   14:34   0:02 /usr/sbin/httpd -DFOREGROUND
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 root 7266 0.0  0.0 221796 2252 pts/1 R+ 20:20   0:00 grep --color=auto http
```

**列出所有安全上下文**

`semanage fcontext -l`

**制作详细规则匹配**

`restorecon 路径`

**配置安全上下文**

`semanage 参数`

```bash
-l 查询
-a 增加
-m 修改
-d 删除
```

**添加上下文**

`semanage fcontext -a -t 上下文 '应用目录 正则表达式'`

```bash
[root@CitProbe web] semanage fcontext -l | grep "^/web"
[root@CitProbe web] **semanage fcontext -a -t httpd_sys_content_t '/web(/.*)?'**
[root@CitProbe web] semanage fcontext -l | grep "^/web"
/web(/.*)?                                         all files          system_u:object_r:httpd_sys_content_t:s0
```

**对此目录重载上下文**

`restorecon 参数 目录`

```bash
-R 递归
-v 详细上下文
```

```bash
[root@CitProbe web] **restorecon -Rv .**
Relabeled /web from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
```

**临时修改上下文**

`chcon -t 上下文 文件路径`

```bash
[root@CitProbe web] ls -lZ
total 0
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 0 Jul 12 21:10 one
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 0 Jul 12 21:10 two
[root@CitProbe web] **chcon -t samba_share_t two**
[root@CitProbe web] ls -lZ
total 0
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 0 Jul 12 21:10 one
-rw-r--r--. 1 root root unconfined_u:object_r:samba_share_t:s0       0 Jul 12 21:10 two
```

**包含semanage的包**

```bash
[root@CitProbe web] rpm -qf /sbin/semanage
**policycoreutils-python-utils-3.5-1.el9.noarch**
```

### SELinux的安全端口

`semanage port -a -t 上下文 -p tcp/udp 端口号`

### 控制SELinux状态

**查看SElinux状态**

`getenforce`

```bash
[root@CitProbe web] getenforce
Enforcing
```

**临时关闭SElinux**

`setenforce 0`

```bash
[root@CitProbe web] setenforce 0
[root@CitProbe web] getenforce
Permissive
```

**临时开启SELinux**

`setenforce 1`

```bash
[root@CitProbe web] setenforce 1
[root@CitProbe web] getenforce
Enforcing
```

**永久修改状态**

配置文件 `/etc/selinux/config`

```bash
# /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-selinux/#getting-started-with-selinux-selinux-states-and-modes
#
# NOTE: In earlier Fedora kernel builds, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing **# 有三种状态 enforcing permissive disabled**
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

<aside>
⚠️ 无论SELinux状态是 Permissive 还是 Enforcing SELinux始终是开启的 Permissive只是不再对访问进行控制 在进行越权访问时 都会被记录在 `/var/log/audit/audit.log` 中

</aside>

## SELinux 布尔值 — 对于服务的访问控制

`semanage boolean -l`

**设置布尔值**

`setsebool -P 布尔值=0/1`