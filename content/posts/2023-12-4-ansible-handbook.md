+++
title = 'Ansible handbook'
date = 2023-12-04T20:28:27+08:00
draft = false

tags = ["ansible",]
categories = ["DevOps"]

+++

# Ansible handbook

## 一、ansible配置

### 1、ansible.cfg

https://ansible-tran.readthedocs.io/en/latest/docs/intro_configuration.html

```
* ANSIBLE_CONFIG (一个环境变量)
* ansible.cfg (位于当前目录中)
* .ansible.cfg (位于家目录中)
* /etc/ansible/ansible.cfg
```

生成ansible的配置文件

```bash
$ ansible-config init --disabled > ~/.ansible.cfg
```

使用环境变量关闭Key验证提示

```bash
$ export ANSIBLE_HOST_KEY_CHECKING=False
```

可使用的模板

```ini
[defaults]
interpreter_python = auto
#interpreter_python = /usr/bin/python3
host_key_checking=False
#inventory=~/.ansible/hosts
private_key_file=/path/to/file.pem
remote_user = root
sudo_user=root
```

### 2、hosts

`写法一`

```ini
node1.ansible.com                 
node2.ansible.com 
    192.168.1.1
```

`写法二`

```ini
[webserver]                            
192.168.10.1
192.168.10.2                

[dbserver]    
192.168.20.1
192.168.20.2
```

注意：如果主机建未做ssh互信，则可以按以下写法

```ini
# 连续IP
[test1]
name1 ansible_ssh_host=192.168.1.[20:50] ansible_ssh_user="root" ansible_ssh_pass="1234" ansible_ssh_port=22 ansible_ssh_private_key_file=~/.ssh/id_rsa


# 带参数
[test1]
name1 ansible_ssh_host=192.168.1.[20:50]
[test1:vars]
ansible_ssh_user=root
ansible_ssh_pass="1234"
testvar="test"
```

host文件常用变量

```ini
ansible_ssh_host     #用于指定被管理的主机的真实IP
ansible_ssh_port     #用于指定连接到被管理主机的ssh端口号，默认是22
ansible_ssh_user     #ssh连接时默认使用的用户名
ansible_ssh_pass     #ssh连接时的密码
ansible_sudo_pass     #使用sudo连接用户时的密码
ansible_sudo_exec     #如果sudo命令不在默认路径，需要指定sudo命令路径
ansible_ssh_private_key_file     #秘钥文件路径，秘钥文件如果不想使用ssh-agent管理时可以使用此选项
ansible_shell_type     #目标系统的shell的类型，默认sh
ansible_connection     #SSH 连接的类型： local , ssh , paramiko，在 ansible 1.2 之前默认是 paramiko ，后来智能选择，优先使用基于 ControlPersist 的 ssh （支持的前提）
ansible_python_interpreter     #用来指定python解释器的路径，默认为/usr/bin/python 同样可以指定ruby 、perl 的路径
ansible_*_interpreter     #其他解释器路径，用法与ansible_python_interpreter类似，这里"*"可以是ruby或才perl等
```



## 二、playbook示例

```yaml
- hosts: rocky
  become: yes
  become_user: root
  roles:
    #- role: serialt.centos_init
    - role: repo
      vars:
        elrepo_install: False
    - role: serialt.os_init
      vars:
        login_info: "        Welcome to local OPS"
        fail2ban_ignore_ip: ""
        ssh_key_message: "t@local.com"
    - role: python  
    - role: serialt.docker
      vars:
        docker_insecure_registries: ["repo.local.com"]
    # - role: serialt.chrony
    #- role: serialt.kernel
   # - role: zabbix-server

```

补充

```shell
# 本地执行
- name: check | 发布文件是否存在。
  shell: "ls {{ deploy_file }}"
  connection: local
  
# 判断
- name: Create software directory.
  file: 
    path: "{{ software_files_path }}"
    state: directory
  connection: local
  when: not go_file_result.stat.exists
  
# 串行执行
- hosts: server1
  become: yes
  gather_facts: yes
  serial: 1
  tasks:
    - YOUR TASKS HERE
```

```shell
- hosts: serialt
  become: yes
  become_user: root
  vars:
    go_version: "1.20.2"
  tasks:
    - name: check | 发布文件是否存在。
      shell: "ls {{ deploy_file }}"
      connection: local
        
```

### 三、公钥受信

`ad hoc`

```shell
ansible add-public-key -m authorized_key -a "user=root state=present key='{{ lookup('file', '/home/root/.ssh/id_rsa.pub') }}'"


```

`playbook`

```yaml
- hosts: add-public-key
  gather_facts: false
  tasks:
  - name: Add local public key to remote hosts.
    authorized_key: 
      user: sugar
      key: "{{ lookup('file', '/Users/serialt/.ssh/id_rsa.pub') }}" 
      state: present


```

`hosts`

```toml
[add-public-key]
172.16.78.53 ansible_connection=ssh  ansible_ssh_user="sugar" ansible_ssh_pass="ubuntu"
; 172.25.70.2 ansible_connection=ssh  ansible_ssh_user="root" ansible_ssh_pass="redhat"
; 172.25.70.3 ansible_connection=ssh  ansible_ssh_user="sugar" ansible_ssh_pass="redhat"
```





## 四、模块简介

### ping

检测被管理端是否在线

`ad-hoc`

```shell
[root@localhost test]# ansible k8s -m ping
192.168.122.100 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```

`playbook`

```yaml | shell
# ping模块没有参数
[root@localhost test]# cat ping.yaml 
- hosts: k8s
  user: root
  gather_facts: false
  tasks:
    - name: test ping
      ping:

```



### shell

在被管理端执行命令，支持重定向和管道

`ad-hoc`

```shell
[root@localhost test]# ansible k8s -m  shell  -a " uptime"
192.168.122.100 | CHANGED | rc=0 >>
 11:11:05 up 25 days,  1:14,  2 users,  load average: 0.31, 0.44, 0.39

# 参数 
chdir=<Directory>
```

`playbook`

```yaml
[root@localhost test]# cat shell.yaml 
- hosts: k8s
  user: root
  gather_facts: false
  tasks:
    - name: test shell
      shell: date && pwd
      args:
        chdir: /tmp/

```

```yaml
    - name: Run expect to wait for a successful PXE boot via out-of-band CIMC
      shell: |
        set timeout 300
        spawn ssh admin@{{ cimc_host }}

        expect "password:"
        send "{{ cimc_password }}\n"

        expect "\n{{ cimc_name }}"
        send "connect host\n"

        expect "pxeboot.n12"
        send "\n"

        exit 0
      args:
        executable: /usr/bin/expect
      delegate_to: localhost

```



### copy

拷贝ansible管理端的文件到远程主机的指定位置

常见参数：

| 参数    | 取值   | 默认值 | 说明                                                                                                        |
| ------- | ------ | ------ | ----------------------------------------------------------------------------------------------------------- |
| dest    | string | null   | 指明拷贝文件的目标目录位置，使用绝对路径，如果源是目录，则目标也要是目录,如果目标文件已存在，会覆盖原有内容 |
| src     | string | null   | 指明本地路径下的某个文件，可以使用相对路径和绝对路径，支持直接指定目录，如果源是目录，则目标也要是目录      |
| mode    | 权限位 | 无     | 指明复制时，目标文件的权限                                                                                  |
| owner   | string | null   | 指明复制时，目标文件的属主                                                                                  |
| group   | string | null   | 指明复制时，目标文件的属组                                                                                  |
| content | string | null   | 指明复制到目标主机上的内容，不能与src一起使用，相当于复制content指明的数据，到目标文件中                    |

```shell
[root@localhost test]# ansible k8s -m copy -a "src=/etc/hosts dest=/tmp/"
192.168.122.100 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "42d592f8ed1579880187c900197e1edd09688992", 
    "dest": "/tmp/hosts", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "f4c82671111086eb6fa6d869e80e1128", 
    "mode": "0644", 
    "owner": "root", 
    "size": 803, 
    "src": "/root/.ansible/tmp/ansible-tmp-1608693775.8-22394-277593855767937/source", 
    "state": "file", 
    "uid": 0
}
```



### fetch

从远程主机拉取文件到本地（ 一般情况下，只会从一个远程节点拉取数据）

常见参数：

| 参数 | 取值   | 默认值 | 说明                                                   |
| ---- | ------ | ------ | ------------------------------------------------------ |
| dest | string | null   | 从远程主机上拉取的文件存放在本地的位置，一般只能是目录 |
| src  | string | null   | 指明远程主机上要拉取的文件，只能是文件，不能是目录     |

```shell
[root@master ~]# ansible test -m fetch -a 'src=/etc/passwd dest=/tmp'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "checksum": "974b44c114ecbd71bdee11e09a9bc14c9b0395bd", 
    "dest": "/tmp/192.168.100.102/etc/passwd", 
    "md5sum": "01d72332a8d9737631212995fe1494f4", 
    "remote_checksum": "974b44c114ecbd71bdee11e09a9bc14c9b0395bd", 
    "remote_md5sum": null
}

```



### cron

管理计划任务

常见参数

| 参数    | 取值              | 默认值0 | 说明                                                                       |
| ------- | ----------------- | ------- | -------------------------------------------------------------------------- |
| minute  | 0-59, * , * / 2   | *       | 指明计划任务的分钟                                                         |
| hour    | 0-23, * , * / 2   | *       |                                                                            |
| day     | 1-31              | *       |                                                                            |
| month   | 1-12              | *       |                                                                            |
| weekday | 0-6               | *       |                                                                            |
| reboot  | yes \| no         | null    | 指明计划任务执行的时间为每次重启之后                                       |
| name    | string            | null    | 给该计划任务取个名称,必须要给明。每个任务的名称不能一样                    |
| job     | string            | null    | 执行的任务是什么，当state=present时才有意义                                |
| state   | present \| absent | present | 表示这个任务是创建还是删除，present表示创建，absent表示删除，默认是present |

```shell
[root@master ~]# ansible test -m cron -a 'minute=*/5 name=Ajob job="/usr/sbin/ntpdate 172.16.8.100 &> /dev/null" state=present'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "Ajob"
    ]
}

[root@master ~]# ansible test -m shell -a 'crontab -l'
192.168.100.102 | SUCCESS | rc=0 >>
#Ansible: Ajob
*/5 * * * * /usr/sbin/ntpdate 172.16.8.100 &> /dev/null

[root@master ~]# ansible test -m cron -a 'minute=*/5 name=Ajob job="/usr/sbin/ntpdate 172.16.8.100 &> /dev/null" state=absent'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "envs": [], 
    "jobs": []
}
```



### file

用于设定远程主机上的文件属性

常见参数:

| 参数  | 取值                       | 默认值 | 说明                                                                            |
| ----- | -------------------------- | ------ | ------------------------------------------------------------------------------- |
| path  | string                     | null   | 指明对哪个文件修改其属性                                                        |
| src   | string                     | null   | 指明path=指明的文件是软链接文件，其对应的源文件是谁，必须要在state=link时才有用 |
| state | directory \| link\| absent | null   | 表示创建的文件是目录还是软链接                                                  |
| owner | string                     | null   | 指明文件的属主                                                                  |
| mode  | 权限位                     | 无     | 指明文件的权限                                                                  |

使用示例：

```shell
创建软链接的用法：
	src=  path=  state=link
修改文件属性的用法：
	path=  owner=  mode=  group=
创建目录的用法：
	path=  state=directory
删除文件：
	path= state=absent
```

```shell
[root@ansible etc]# ansible testsrv -m file -a "path=/tmp/1.txt mode=600 owner=root group=nobody"

[root@ansible ~]# ansible testsrv -m file -a "path=/tmp/bb mode=777 recurse=yes"
```

创建软链接

```shell
[root@master ~]# ansible test -m file -a 'src=/etc/passwd path=/tmp/passwd.link state=link'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "dest": "/tmp/passwd.link", 
    "gid": 0, 
    "group": "root", 
    "mode": "0777", 
    "owner": "root", 
    "size": 11, 
    "src": "/etc/passwd", 
    "state": "link", 
    "uid": 0
}

```

删除文件

```shell
[root@master ~]# ansible test -m file -a 'path=/tmp/cc.txt state=absent'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "path": "/tmp/cc.txt", 
    "state": "absent"
}
```

修改文件属性

```shell
[root@master ~]# ansible test -m file -a 'path=/tmp/bb.txt mode=700 owner=root group=nobody'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "gid": 99, 
    "group": "nobody", 
    "mode": "0700", 
    "owner": "root", 
    "path": "/tmp/bb.txt", 
    "size": 14, 
    "state": "file", 
    "uid": 0
}
[root@master ~]# ansible test -m shell -a 'ls -l /tmp/bb.txt'
192.168.100.102 | SUCCESS | rc=0 >>
-rwx------ 1 root nobody 14 Dec  2  2016 /tmp/bb.txt

```

创建目录

```shell
[root@master ~]# ansible test -m file -a 'path=/tmp/bj state=directory'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/tmp/bj", 
    "size": 4096, 
    "state": "directory", 
    "uid": 0
}

```

删除目录

```shell
[root@master ~]# ansible test -m file -a 'path=/tmp/bj state=absent'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "path": "/tmp/bj", 
    "state": "absent"
}
[root@master ~]# ansible test -m shell -a 'ls -l /tmp'
192.168.100.102 | SUCCESS | rc=0 >>
total 16
-rw-r--r-- 1 root   root      0 Dec  2  2016 aa.txt
drwx------ 2 root   root   4096 Dec  2 13:41 ansible_twMJYb
-rwx------ 1 root   nobody   14 Dec  2  2016 bb.txt
-rw-r--r-- 1 root   root    158 Dec  2  2016 hosts
-rw------- 1 nobody nobody  947 Dec  2  2016 passwd
lrwxrwxrwx 1 root   root     11 Dec  2 13:35 passwd.link -> /etc/passwd
-rw------- 1 root   root      0 Dec  2 00:58 yum.log
```



### hostsname

管理远程主机的主机名

参数：

* name=  指明主机名

```shell
[root@master ~]# ansible test -m shell -a 'hostname'
192.168.100.102 | SUCCESS | rc=0 >>
node1.ansible.com

[root@master ~]# ansible test -m hostname -a 'name=node2.ansible.com'
192.168.100.102 | SUCCESS => {
    "ansible_facts": {
        "ansible_domain": "ansible.com", 
        "ansible_fqdn": "node2.ansible.com", 
        "ansible_hostname": "node2", 
        "ansible_nodename": "node2.ansible.com"
    }, 
    "changed": true, 
    "name": "node2.ansible.com"
}
```



### yum

基于yum机制，对远程主机管理程序包

常用参数：

| 参数              | 取值                     | 默认值 | 说明                                                                                                |
| ----------------- | ------------------------ | ------ | --------------------------------------------------------------------------------------------------- |
| name              | string                   | null   | 指明程序包的名称，可以带上版本号，不指明版本，就是默认最新版本                                      |
| state             | present\|lastest\|absent | null   | 指明对程序包执行的操作，present表示安装程序包，latest表示安装最新版本的程序包，absent表示卸载程序包 |
| disablerepo       | yes\|no                  | null   | 在用yum安装时，临时禁用某个仓库，仓库的ID                                                           |
| enablerepo        | yes\|no                  | null   | 在用yum安装时，临时启用某个仓库,仓库的ID                                                            |
| conf_file=        | string                   | null   | 指明yum运行时采用哪个配置文件，而不是使用默认的配置文件                                             |
| disable_gpg_check | yes\|no                  | null   | 是否启用gpg-check                                                                                   |

```shell
# 卸载软件包:
[root@master ~]# ansible test -m yum -a 'name=httpd state=absent'
[root@master ~]# ansible test -m shell -a 'rpm -q httpd'


# 安装软件包:
[root@master ~]# ansible test -m yum -a 'name=httpd state=present'

[root@ansible ~]# ansible 192.168.122.102 -m yum -a "name=ftp state=present disablerepo=zabbix"

[root@ansible_server ~]# ansible test_server -m yum -a "name=zabbix-agent state=present enablerepo=zabbix3.2 disablerepo=zabbix"


# 更新软件
[root@ansible_server ~]# ansible test_server -m yum -a "name=zabbix-agent state=latest"

```

`palybook`

```yaml
- hosts: all
  tasks:
  - name: yum
    yum:
      name: "{{ item }}"
      state: present
    with_items:
    - git
    - httpd
    - mysql
```



### yum_repository

管理远程主机上的 yum 仓库

常用参数：

| 参数        | 取值    | 默认值  | 说明                                                                                                                                                                                        |
| ----------- | ------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name        | string  | null    | 必须参数，用于指定要操作的唯一的仓库ID，也就是”.repo”配置文件中每个仓库对应的”中括号”内的仓库ID                                                                                             |
| baseurl     | string  | null    | 用于设置 yum 仓库的 baseurl                                                                                                                                                                 |
| description | string  | null    | 用于设置仓库的注释信息，也就是”.repo”配置文件中每个仓库对应的”name字段”对应的内容                                                                                                           |
| file        | string  | null    | 用于设置仓库的配置文件名称，即设置”.repo”配置文件的文件名前缀，在不使用此参数的情况下，默认以 name 参数的仓库ID作为”.repo”配置文件的文件名前缀，同一个”.repo” 配置文件中可以存在多个 yum 源 |
| enabled     | string  | yes     | 用于设置是否激活对应的 yum 源，此参数默认值为 yes，表示启用对应的 yum 源，设置为 no 表示不启用对应的 yum 源                                                                                 |
| gpgcheck    | string  | no      | 用于设置是否开启 rpm 包验证功能，默认值为 no，表示不启用包验证，设置为 yes 表示开启包验证功能                                                                                               |
| gpgcakey    | string  | null    | 当 gpgcheck 参数设置为 yes 时，需要使用此参数指定验证包所需的公钥                                                                                                                           |
| state       | present | present | 当值设置为 absent 时，表示删除对应的 yum 源                                                                                                                                                 |

```shell
[root@ansible-manager ~]# ansible ansible-demo3 -m yum_repository -a 'name=aliEpel description="alibaba EPEL" baseurl=https://mirrors.aliyun.com/epel/$releasever\Server/$basearch/'
ansible-demo3 | SUCCESS => {
    "changed": true, 
    "repo": "aliEpel", 
    "state": "present"
}

```

配置postgresql 清华源

```shell
[root@localhost test]# cat yum_repo.yaml 
- hosts: 127.0.0.1
  user: root
  gather_facts: false
  tasks:
    - name: config postgresql with tsinghua
      yum_repository:
        file: postgresql-10
        name: postgresql
        description: postgresql-10 tsinghua
        baseurl: https://mirrors.tuna.tsinghua.edu.cn/postgresql/repos/yum/10/redhat/rhel-7-x86_64/
        enabled: yes
        gpgcheck: no

[root@localhost yum.repos.d]# cat postgresql-10.repo 
[postgresql]
baseurl = https://mirrors.tuna.tsinghua.edu.cn/postgresql/repos/yum/10/redhat/rhel-7-x86_64/
enabled = 1
gpgcheck = 0
name = postgresql-10 tsinghua

```





### service

管理远程主机上的服务的模块

常用参数：

| 参数    | 取值                        | 默认值  | 说明                          |
| ------- | --------------------------- | ------- | ----------------------------- |
| name    | string                      | null    | 被管理的服务名称(/etc/init.d) |
| state   | started\|stopped\|restarted | started | 启动或关闭或重起              |
| enabled | yes\|no                     | no      | 设定该服务开机自启动          |

```shell
[root@master ~]# ansible test -m service -a 'name=nginx state=started'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "name": "nginx", 
    "state": "started"
}
[root@master ~]# ansible test -m shell -a 'service nginx status'
192.168.100.102 | SUCCESS | rc=0 >>
nginx (pid  4054) is running...

[root@master ~]# 


[root@master ~]# ansible test -m service -a 'name=nginx state=stopped'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "name": "nginx", 
    "state": "stopped"
}
[root@master ~]# ansible test -m shell -a 'service nginx status'
192.168.100.102 | FAILED | rc=3 >>
nginx is stopped


[root@master ~]# ansible test -m service -a 'name=nginx state=started enabled=yes runlevel=2345'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "enabled": true, 
    "name": "nginx", 
    "state": "started"
}
[root@master ~]# ansible test -m shell -a 'chkconfig --list nginx'
192.168.100.102 | SUCCESS | rc=0 >>
nginx          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```



### systemd

管理被控制主机的systemd服务

常见参数：

| 参数          | 取值                                  | 默认值 | 说明                                |
| ------------- | ------------------------------------- | ------ | ----------------------------------- |
| daemon_reload | yes\|no                               | 无     | 重启守护进程，与其他systemd参数排斥 |
| name          | string                                | null   | 被管理的服务名                      |
| enabled       | yes\|no                               | null   | 服务设置为开机自启                  |
| state         | reloaded\|restarted\|started\|stopped | null   | 服务状态                            |

```shell
[root@localhost test]# cat daemon_reload.yaml 
- hosts: 127.0.0.1
  user: root
  gather_facts: false
  tasks:
    - name: daemon reload
      systemd:
        daemon_reload: yes

[root@localhost test]# cat nginx.yaml 
- hosts: 127.0.0.1
  user: root
  gather_facts: false
  tasks:
    - name: daemon reload
      systemd:
        name: nginx
        enabled: yes
        state: restarted

```



### git

管理git服务模块

常见参数：

| 参数    | 取值    | 默认值 | 说明                                                       |
| ------- | ------- | ------ | ---------------------------------------------------------- |
| bare    | yes\|no | no     | 建立裸仓库，用于服务器端git仓库存储                        |
| clone   | yes\|no | yes    | 本地镜像仓库不存在则clone                                  |
| depth   | 数字    | null   | clone时克隆的commit深度，1表示只克隆最新一次提交，默认null |
| dest    | string  | null   | clone出的仓库存储的路径，仅当clone=no时不用设置            |
| force   | yes\|no | no     | 强制                                                       |
| remote  | string  | null   | 远程仓库的名称，默认是origin                               |
| repo    | string  | null   | 远程仓库的地址                                             |
| version | string  | null   | checkout的版本，支持分支、tag、hash                        |
| archive | string  | null   | clone后的压缩文件存储文件，支持格式zip、tar.gz、tar、tgz   |

```
- hosts: 127.0.0.1
  user: root
  gather_facts: false
  vars:
    - GIT_REPO: git@serialt.io:sugars/sugars_backend.git
    - SRC_HASH: 4949e0b60be3f6a80826d3e02exxxxxxxxxxxx
  tasks:
    - name: clone a git repo 
      git:
        repo: "{{ item.repo }}" 
        version: "{{ item.hash }}"   
        dest: /tmp/serialt_backend
      with_items:
        - {repo: "{{ GIT_REPO }}",hash: "{{ SRC_HASH }}" }
```



### supervisorctl

管理supervisorctl服务

常见参数：


| 参数               | 取值   | 默认值 | 说明                                                        |
| ------------------ | ------ | ------ | ----------------------------------------------------------- |
| config             | string | null   | supervisor配置文件路径                                      |
| name               | string | null   | supervisord program的名字，支持program和program组           |
| state              | string | nul    | 可选present, started, stopped, restarted, absent, signalled |
| supervisorctl_path | string | null   | supervisorctl命令的路径                                     |

```yaml
EXAMPLES:

# Manage the state of program to be in 'started' state.
- supervisorctl:
    name: my_app
    state: started

# Manage the state of program group to be in 'started' state.
- supervisorctl:
    name: 'my_apps:'
    state: started

# Restart my_app, reading supervisorctl configuration from a specified file.
- supervisorctl:
    name: my_app
    state: restarted
    config: /var/opt/my_project/supervisord.conf

# Restart my_app, connecting to supervisord with credentials and server URL.
- supervisorctl:
    name: my_app
    state: restarted
    username: test
    password: testpass
    server_url: http://localhost:9001

# Send a signal to my_app via supervisorctl
- supervisorctl:
    name: my_app
    state: signalled
    signal: USR1
```





### pip

管理python库依赖模块

常见参数：


| 参数               | 取值   | 默认值  | 说明                                                               |
| ------------------ | ------ | ------- | ------------------------------------------------------------------ |
| chdir              | string | null    | 执行时的路径                                                       |
| name               | string | null    | pip安装的包名                                                      |
| requirements       | string | null    | 安装依赖的包文件                                                   |
| executable         | string | null    | 安装依赖包时使用的python路径                                       |
| state              | string | present | 依赖包的状态，absent, forcereinstall, latest, present，默认present |
| virtualenv         | string | null    | 所使用的虚拟环境目录                                               |
| virtualenv_command | string | null    | 创建虚拟环境的命令                                                 |
| virtualenv_python  | string | null    | 虚拟环境的python版本                                               |
| extra_args=        | string | null    | 附加参数，可以用来指定所使用的安装源                               |

```
[root@localhost test]# cat pip.yaml 
- hosts: 192.168.100.105
  user: root
  gather_facts: false
  tasks:
    - name: install python venv 
      pip:
        requirements: /tmp/requirements.txt 
        state: present
        virtualenv: /tmp/serialt_venv/

```



### archive 

压缩文件模块

常用参数：


| 参数         | 取值   | 默认值 | 说明                                                                        |
| ------------ | ------ | ------ | --------------------------------------------------------------------------- |
| dest         | string | null   | 文件归档后的压缩包文件名，当 `path`中有多个文件或目录时，需提供 `dest` 参数 |
| exclude_path | string | null   | 排除的path                                                                  |
| format       | string | gz     | 支持bz2, gz, tar, xz, zip，默认gz                                           |
| path         | string | null   | 要压缩的路径                                                                |
| remove       | bool   | False  | 删除源文件                                                                  |

```yaml
EXAMPLES:

- name: Compress directory /path/to/foo/ into /path/to/foo.tgz
  archive:
    path: /path/to/foo
    dest: /path/to/foo.tgz

- name: Compress regular file /path/to/foo into /path/to/foo.gz and remove it
  archive:
    path: /path/to/foo
    remove: yes

- name: Create a zip archive of /path/to/foo
  archive:
    path: /path/to/foo
    format: zip

- name: Create a bz2 archive of multiple files, rooted at /path
  archive:
    path:
    - /path/to/foo
    - /path/wong/foo
    dest: /path/file.tar.bz2
    format: bz2

- name: Create a bz2 archive of a globbed path, while excluding specific dirnames
  archive:
    path:
    - /path/to/foo/*
    dest: /path/file.tar.bz2
    exclude_path:
    - /path/to/foo/bar
    - /path/to/foo/baz
    format: bz2

- name: Create a bz2 archive of a globbed path, while excluding a glob of dirnames
  archive:
    path:
    - /path/to/foo/*
    dest: /path/file.tar.bz2
    exclude_path:
    - /path/to/foo/ba*
    format: bz2

- name: Use gzip to compress a single archive (i.e don't archive it first with tar)
  archive:
    path: /path/to/foo/single.file
    dest: /path/file.gz
    format: gz

- name: Create a tar.gz archive of a single file.
  archive:
    path: /path/to/foo/single.file
    dest: /path/file.tar.gz
    format: gz
    force_archive: true


RETURN VALUES:

- name: Create a zip archive of /path/to/foo
  archive:
    path: /path/to/foo
    format: zip

- name: Create a bz2 archive of multiple files, rooted at /path
  archive:
    path:
    - /path/to/foo
    - /path/wong/foo
    dest: /path/file.tar.bz2
    format: bz2

- name: Create a bz2 archive of a globbed path, while excluding specific dirnames
  archive:
    path:
    - /path/to/foo/*
    dest: /path/file.tar.bz2
    exclude_path:
    - /path/to/foo/bar
    - /path/to/foo/baz
    format: bz2

- name: Create a bz2 archive of a globbed path, while excluding a glob of dirnames
  archive:
    path:
    - /path/to/foo/*
    dest: /path/file.tar.bz2
    exclude_path:
    - /path/to/foo/ba*
    format: bz2

- name: Use gzip to compress a single archive (i.e don't archive it first with tar)
  archive:
    path: /path/to/foo/single.file
    dest: /path/file.gz
    format: gz

- name: Create a tar.gz archive of a single file.
  archive:
    path: /path/to/foo/single.file
    dest: /path/file.tar.gz
    format: gz
    force_archive: true
```



### unarchive

| 参数    | 取值     | 默认值 | 说明                                                                                                    |
| ------- | -------- | ------ | ------------------------------------------------------------------------------------------------------- |
| creates | string   | null   | 文件名，当它已经存在时，这个步骤将不会被运行                                                            |
| copy    | yes / no | yes    | 拷贝的文件从ansible主机复制到远程主机，no在远程主机上寻找src源文件解压                                  |
| src     | string   | null   | tar源路径，可以是ansible主机上的路径，也可以是远程主机上的路径，如果是远程主机上的路径，则需设置copy=no |
| dest    | string   | null   | 远程主机上的目标绝对路径                                                                                |
| mode    | 数字     | 无     | 设置解压缩后的文件权限                                                                                  |
| exec    | 无       | 无     | 列出需要排除的目录和文件                                                                                |
| owner   | string   | 无     | 解压后文件或目录的属主                                                                                  |
| group   | srting   | 无     | 解压后的目录或文件的属组                                                                                |


```yaml
EXAMPLES:

- name: Extract foo.tgz into /var/lib/foo
  unarchive:
    src: foo.tgz
    dest: /var/lib/foo

- name: Unarchive a file that is already on the remote machine
  unarchive:
    src: /tmp/foo.zip
    dest: /usr/local/bin
    remote_src: yes

- name: Unarchive a file that needs to be downloaded (added in 2.0)
  unarchive:
    src: https://example.com/example.zip
    dest: /usr/local/bin
    remote_src: yes

- name: Unarchive a file with extra options
  unarchive:
    src: /tmp/foo.zip
    dest: /usr/local/bin
    extra_opts:
    - --transform
    - s/^xxx/yyy/
```

### uri

用于请求某个网页

常见参数：

| 参数             | 取值   | 默认值 | 说明                                           |
| ---------------- | ------ | ------ | ---------------------------------------------- |
| body             | raw    | null   | 当body_format设置为json时的键值对              |
| body_format      | string | raw    | 数据格式，form-urlencoded, json, raw           |
| method           | string | GET    | 指明请求的方法，如GET、POST, PUT, DELETE, HEAD |
| dest             | string | null   | 下载文件的路径                                 |
| force            | bool   | False  |                                                |
| force_basic_auth |        |        |                                                |
| headers          |        |        |                                                |
| method           |        |        | 指明请求的方法，如GET、POST, PUT, DELETE, HEAD |
| return_content   | bool   | False  |                                                |
| url              | string | null   |                                                |
| url_password     |        |        | 如果请求的url需要认证，则填写认证的密码        |
| url_username     |        |        | 如果请求的url需要认证，则填写用户名            |



```
[root@master ~]# ansible test -m url -a 'url=http://192.168.100.102/index.html'
192.168.100.102 | SUCCESS => {
    "accept_ranges": "bytes", 
    "changed": false, 
    "connection": "close", 
    "content_length": "612", 
    "content_type": "text/html", 
    "date": "Fri, 02 Dec 2016 06:31:58 GMT", 
    "etag": "\"571f8501-264\"", 
    "last_modified": "Tue, 26 Apr 2016 15:10:57 GMT", 
    "msg": "OK (612 bytes)", 
    "redirected": false, 
    "server": "nginx/1.10.0", 
    "status": 200, 
    "url": "http://192.168.100.102/index.html"
}
[root@master ~]# 
```

```yaml
EXAMPLES:

- name: Check that you can connect (GET) to a page and it returns a status 200
  uri:
    url: http://www.example.com

- name: Check that a page returns a status 200 and fail if the word AWESOME is not in the page contents
  uri:
    url: http://www.example.com
    return_content: yes
  register: this
  failed_when: "'AWESOME' not in this.content"

- name: Create a JIRA issue
  uri:
    url: https://your.jira.example.com/rest/api/2/issue/
    user: your_username
    password: your_pass
    method: POST
    body: "{{ lookup('file','issue.json') }}"
    force_basic_auth: yes
    status_code: 201
    body_format: json

- name: Login to a form based webpage, then use the returned cookie to access the app in later tasks
  uri:
    url: https://your.form.based.auth.example.com/index.php
    method: POST
    body_format: form-urlencoded
    body:
      name: your_username
      password: your_password
      enter: Sign in
    status_code: 302
  register: login

- name: Login to a form based webpage using a list of tuples
  uri:
    url: https://your.form.based.auth.example.com/index.php
    method: POST
    body_format: form-urlencoded
    body:
    - [ name, your_username ]
    - [ password, your_password ]
    - [ enter, Sign in ]
    status_code: 302
  register: login

- name: Connect to website using a previously stored cookie
  uri:
    url: https://your.form.based.auth.example.com/dashboard.php
    method: GET
    return_content: yes
    headers:
      Cookie: "{{ login.set_cookie }}"

- name: Queue build of a project in Jenkins
  uri:
    url: http://{{ jenkins.host }}/job/{{ jenkins.job }}/build?token={{ jenkins.token }}
    user: "{{ jenkins.user }}"
    password: "{{ jenkins.password }}"
    method: GET
    force_basic_auth: yes
    status_code: 201

- name: POST from contents of local file
  uri:
    url: https://httpbin.org/post
    method: POST
    src: file.json

- name: POST from contents of remote file
  uri:
    url: https://httpbin.org/post
    method: POST
    src: /path/to/my/file.json
    remote_src: yes

- name: Pause play until a URL is reachable from this host
  uri:
    url: "http://192.0.2.1/some/test"
    follow_redirects: none
    method: GET
  register: _result
  until: _result.status == 200
  retries: 720 # 720 * 5 seconds = 1hour (60*60/5)
  delay: 5 # Every 5 seconds

# There are issues in a supporting Python library that is discussed in
# https://github.com/ansible/ansible/issues/52705 where a proxy is defined
# but you want to bypass proxy use on CIDR masks by using no_proxy
- name: Work around a python issue that doesn't support no_proxy envvar
  uri:
    follow_redirects: none
    validate_certs: false
    timeout: 5
    url: "http://{{ ip_address }}:{{ port | default(80) }}"
  register: uri_data
  failed_when: false
  changed_when: false
  vars:
    ip_address: 192.0.2.1
  environment: |
      {
        {% for no_proxy in (lookup('env', 'no_proxy') | regex_replace('\s*,\s*', ' ') ).split() %}
          {% if no_proxy | regex_search('\/') and
                no_proxy | ipaddr('net') != '' and
                no_proxy | ipaddr('net') != false and
                ip_address | ipaddr(no_proxy) is not none and
                ip_address | ipaddr(no_proxy) != false %}
            'no_proxy': '{{ ip_address }}'
          {% elif no_proxy | regex_search(':') != '' and
                  no_proxy | regex_search(':') != false and
                  no_proxy == ip_address + ':' + (port | default(80)) %}
            'no_proxy': '{{ ip_address }}:{{ port | default(80) }}'
          {% elif no_proxy | ipaddr('host') != '' and
                  no_proxy | ipaddr('host') != false and
                  no_proxy == ip_address %}
            'no_proxy': '{{ ip_address }}'
          {% elif no_proxy | regex_search('^(\*|)\.') != '' and
                  no_proxy | regex_search('^(\*|)\.') != false and
                  no_proxy | regex_replace('\*', '') in ip_address %}
            'no_proxy': '{{ ip_address }}'
          {% endif %}
        {% endfor %}
      }
```



### group

用来添加或删除远端主机的用户组

常见参数：

| 参数   | 取值   | 默认值  | 说明         |
| ------ | ------ | ------- | ------------ |
| name   | string | null    |              |
| state  | string | present |              |
| gid    | int    | null    | GID          |
| system | bool   | False   | 创建系统用户 |

```
[root@master ~]# ansible test -m group -a 'name=hr gid=2000 state=present'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "gid": 2000, 
    "name": "hr", 
    "state": "present", 
    "system": false
}

[root@master ~]# ansible test -m shell -a 'tail -1 /etc/group'
192.168.100.102 | SUCCESS | rc=0 >>
hr:x:2000:
```



### user

管理远程主机上的用户的账号

常见参数：

| 参数      | 取值   | 默认值  | 说明                                                                       |
| --------- | ------ | ------- | -------------------------------------------------------------------------- |
| name      | string | null    | 指明要管理的账号名称                                                       |
| state     | string | present | 指明是创建账号还是删除账号，present表示创建，absent表示删除                |
| system    | bool   | False   | 指定系统用户                                                               |
| uid       | int    | null    | 用户的uid                                                                  |
| shell     | string | null    | shell类型                                                                  |
| home      | string | null    | 家目录位置                                                                 |
| group     | string | null    | 指明用户的基本组                                                           |
| groups    | string | null    | 指明用户的附加组                                                           |
| move_home | bool   | False   | 当home设定了家目录，如果要创建的家目录已存在，是否将已存在的家目录进行移动 |
| password  | string | null    | 指明用户的密码，最好使用加密好的字符串                                     |
| comment   | string | null    | 指明用户的注释信息                                                         |
| remove    | bool   | null    | 当state=absent时，也就是删除用户时，是否要删除用户的而家目录               |

```
[root@master ~]# ansible test -m user -a 'name=martin group=hr groups=shichang uid=500 shell=/bin/bash home=/home/martin comment="martin user"'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "comment": "martin user", 
    "createhome": true, 
    "group": 2000, 
    "groups": "shichang", 
    "home": "/home/martin", 
    "name": "martin", 
    "shell": "/bin/bash", 
    "state": "present", 
    "system": false, 
    "uid": 500
}
[root@master ~]# ansible test -m shell -a 'grep "martin:" /etc/passwd'
192.168.100.102 | SUCCESS | rc=0 >>
martin:x:500:2000:martin user:/home/martin:/bin/bash


[root@master ~]# ansible test -m user -a 'name=martin state=absent remove=yes'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "force": false, 
    "name": "martin", 
    "remove": true, 
    "state": "absent"
}

```

```
EXAMPLES:

- name: Add the user 'johnd' with a specific uid and a primary group of 'admin'
  user:
    name: johnd
    comment: John Doe
    uid: 1040
    group: admin

- name: Add the user 'james' with a bash shell, appending the group 'admins' and 'developers' to the user's gro
  user:
    name: james
    shell: /bin/bash
    groups: admins,developers
    append: yes

- name: Remove the user 'johnd'
  user:
    name: johnd
    state: absent
    remove: yes

- name: Create a 2048-bit SSH key for user jsmith in ~jsmith/.ssh/id_rsa
  user:
    name: jsmith
    generate_ssh_key: yes
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_rsa

- name: Added a consultant whose account you want to expire
  user:
    name: james18
    shell: /bin/zsh
    groups: developers
    expires: 1422403387

- name: Starting at Ansible 2.6, modify user, remove expiry time
  user:
    name: james18
    expires: -1
```



### script

将管理端的某个脚本，移动到远端主机(不需要指明传递到远端主机的哪个路径下，系统会自动移动，然后执行)，         一般是自动移动到远端主机的/root/.ansible/tmp目录下，然后自动给予其权限，然后再开个子shell然后运行脚本，运行完成后删除脚本

```
[root@master ~]# ansible test -m script -a '/root/1.sh'
192.168.100.102 | SUCCESS => {
    "changed": true, 
    "rc": 0, 
    "stderr": "", 
    "stdout": "", 
    "stdout_lines": []
}
[root@master ~]#
```



### setup

   可收集远程主机的facts变量的信息，相当于收集了目标主机的相关信息(如内核版本、操作系统信息、cpu、…)，保存在ansible的内置变量中，之后我们有需要用到时，直接调用变量即可

```
[root@master ~]# ansible test -m setup
192.168.100.102 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.100.102"
        ], 
        "ansible_all_ipv6_addresses": [
            "fe80::20c:29ff:fe0c:5ab9"
        ], 
        "ansible_architecture": "x86_64", 
        "ansible_bios_date": "05/20/2014",  
        "ansible_bios_version": "6.00", 
```



### template

基于模板方式，生成一个模板文件，复制到远程主机，让远程主机基于模板，生成符合远程主机自身的文件

注意：此模块不能在命令行使用，只能用在playbook中

常见参数：

* src=          指明管理端本地的模板文件的目录
* dest=          指明将模板文件拷贝到远程主机的哪个目录下
* owner=      指明拷贝到远程主机的文件的属主
* group=          指明拷贝到远程主机的文件的属组
* mode=     指明拷贝到远程主机的文件的权限

```
[root@master ~]# cat temp.txt 
this is {{ ansible_hostname  }}    

[root@master ~]# cat test.yml 
- hosts: 192.168.10.202
  remote_user: root
  tasks:
  - name: test template 
    template: src=/root/temp.txt dest=/tmp

[root@master ~]# ansible-playbook test.yml 

PLAY [192.168.10.202] **********************************************************

TASK [setup] *******************************************************************
ok: [192.168.10.202]

TASK [test template module] ****************************************************
changed: [192.168.10.202]

PLAY RECAP *********************************************************************
192.168.10.202             : ok=2    changed=1    unreachable=0    failed=0   

[root@master ~]# ansible 192.168.10.202 -m shell -a 'cat /tmp/temp.txt'
192.168.10.202 | SUCCESS | rc=0 >>
this is agent202

```



### script

script 模块可以在远程主机上执行 ansible 管理主机上的脚本，也就是说，脚本一直存在于 ansible 管理主机本地，不需要手动拷贝到远程主机后再执行。

常用参数：

* chdir：执行脚本时所在的目录
* creates：使用此参数指定一个远程主机中的文件，当指定的文件存在时，就不执行对应脚本
* removes：使用此参数指定一个远程主机中的文件，当指定的文件不存在时，就不执行对应脚本



示例：

```
- name: Run a script with arguments (free form)
  script: /some/local/script.sh
```



### lineinfile

| 参数         | 取值            | 默认值  | 说明           |
| ------------ | --------------- | ------- | -------------- |
| path         | string          | null    | 操作的文件     |
| line         | string          | null    | 替换的内容     |
| regexp       | string          | null    | 匹配表达式     |
| state        | present\|absent | present |                |
| insertafter  | string          | 无      | 插入到匹配行后 |
| insertbefore | string          | 无      | 插入匹配行前   |
| backup       | yes\|no         | 无      | 改变前备份     |
|              |                 |         |                |

示例：开启selinux

```
- name: Ensure SELinux is set to enforcing mode
  lineinfile:
    path: /etc/selinux/config
    regexp: '^SELINUX='
    line: SELINUX=enforcing

```

```
- name: Make sure group wheel is not in the sudoers configuration
  lineinfile:
    path: /etc/sudoers
    state: absent
    regexp: '^%wheel'

```

```
- name: Replace a localhost entry with our own
  lineinfile:
    path: /etc/hosts
    regexp: '^127\.0\.0\.1'
    line: 127.0.0.1 localhost
    owner: root
    group: root
    mode: '0644'

```



### fail

| 参数 | 取值   | 默认值 | 说明                     |
| ---- | ------ | ------ | ------------------------ |
| msg  | string | null   | 满足条件执行时输出的信息 |

```
- fail:
    msg: The system may not be provisioned according to the CMDB status.
  when: cmdb_status != "to-be-staged"

```



### stat

检查文件或文件系统的状态，对于Windows目标，则使用win_stat模块

| 参数               | 取值   | 默认值 | 说明                                                          |
| ------------------ | ------ | ------ | ------------------------------------------------------------- |
| path               | string | null   | 文件或目录的路径，必选参数                                    |
| checksum_algorithm | string | sha1   | 计算文件的算法，可选md5, sha1, sha224, sha256, sha384, sha512 |

stat模块的返回值

| 返回值     | 取值 |
| ---------- | ---- |
| exists     | bool |
| path       | str  |
| mode       | str  |
| isdir      | bool |
| islnk      | bool |
| uid        | int  |
| gid        | int  |
| size       | int  |
| inode      | int  |
| lnk_source | str  |
| md5        | str  |



使用示例

```yaml
- stat:
    path: /etc/foo.conf
  register: st
- fail:
    msg: "Whoops! file ownership has changed"
  when: st.stat.pw_name != 'root'
```

```yaml
- stat:
    path: /path/to/something
  register: sym

- debug:
    msg: "islnk isn't defined (path doesn't exist)"
  when: sym.stat.islnk is not defined

- debug:
    msg: "islnk is defined (path must exist)"
  when: sym.stat.islnk is defined

- debug:
    msg: "Path exists and is a symlink"
  when: sym.stat.islnk is defined and sym.stat.islnk

- debug:
    msg: "Path exists and isn't a symlink"
  when: sym.stat.islnk is defined and sym.stat.islnk == False

```

```yaml
- stat:
    path: /path/to/something
  register: p
- debug:
    msg: "Path exists and is a directory"
  when: p.stat.isdir is defined and p.stat.isdir

# Don't do checksum
- stat:
    path: /path/to/myhugefile
    get_checksum: no

# Use sha256 to calculate checksum
- stat:
    path: /path/to/something
    checksum_algorithm: sha256

```





### pause

暂停一段时间等待终端输入

| 参数    | 取值   | 默认值 | 说明                     |
| ------- | ------ | ------ | ------------------------ |
| echo    | bool   | yes    | 是否输出键盘的输入值     |
| minutes | string | null   | 暂停多少分钟             |
| prompt  | string | null   | 打印一串信息提示用户操作 |
| seconds | string | null   | 暂停多少秒               |
|         |        |        |                          |
|         |        |        |                          |

pause模块的返回值

| 返回值     | 取值   |
| ---------- | ------ |
| delta      | string |
| echo       | bool   |
| start      | string |
| stdout     | string |
| stop       | string |
| user_input | string |

使用示例

```yaml
- name: Pause for 5 minutes to build app cache
  pause:
    minutes: 5

- name: Pause until you can verify updates to an application were successful
  pause:

- name: A helpful reminder of what to look out for post-update
  pause:
    prompt: "Make sure org.foo.FooOverload exception is not present"

- name: Did you Backup DB
  pause: prompt="Did you commit it? Enter to continue or CTRL-C to quit."

- name: Pause to get some sensitive input
  pause:
    prompt: "Enter a secret"
    echo: no
```



