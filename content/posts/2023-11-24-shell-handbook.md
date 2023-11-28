+++

title = 'Shell handbook'
date = 2023-11-24T20:22:27+08:00
draft = false

tags = ["shell","shell-handbook"]
categories = ["DevOps"]

+++



# Shell handbok



## 一、特殊符号

参考链接:

* https://blog.csdn.net/jiezi2016/article/details/79649382 
* https://blog.csdn.net/wangzhaotongalex/article/details/73321766

```shell
介绍下Shell中的${}、##和%%使用范例，本文给出了不同情况下得到的结果。
假设定义了一个变量为：
代码如下:
file=/dir1/dir2/dir3/my.file.txt
可以用${ }分别替换得到不同的值：
${file#*/}：删掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
${file##*/}：删掉最后一个 /  及其左边的字符串：my.file.txt
${file#*.}：删掉第一个 .  及其左边的字符串：file.txt
${file##*.}：删掉最后一个 .  及其左边的字符串：txt
${file%/*}：删掉最后一个  /  及其右边的字符串：/dir1/dir2/dir3
${file%%/*}：删掉第一个 /  及其右边的字符串：(空值)
${file%.*}：删掉最后一个  .  及其右边的字符串：/dir1/dir2/dir3/my.file
${file%%.*}：删掉第一个  .   及其右边的字符串：/dir1/dir2/dir3/my
记忆的方法为：
# 是 去掉左边（键盘上#在 $ 的左边）
%是去掉右边（键盘上% 在$ 的右边）
单一符号是最小匹配；两个符号是最大匹配
${file:0:5}：提取最左边的 5 个字节：/dir1
${file:5:5}：提取第 5 个字节右边的连续5个字节：/dir2
也可以对变量值里的字符串作替换：
${file/dir/path}：将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt
${file//dir/path}：将全部dir 替换为 path：/path1/path2/path3/my.file.txt
```

`$*`与`$@`

$*和$@都表示传递给函数或脚本的所有参数，不被双引号“”包含时，都以$1 $2 …$n的形式输出所有参数。

当它们被双引号“”包含时，“$*”会将所有的参数作为一个整体，以“$1 $2 …$n”的形式输出所有参数；“$@”会将各个参数分开，以“$1” “$2”…”$n”的形式输出所有参数。



## 二、重定向与变量

```shell
cat >>test.txt << EOF
1、安装kvm
2、卸载kvm
3、安装python
EOF
```

```shell
$0	shell脚本的名字
$n	第n个参数
$#  获取参数的个数
$*	所有参数
$@	所有参数
	两者区别：
	"$*"会把所有位置参数当成一个整体（或者说当成一个单词），如果没有位置参数，则"$*"为空，如果有两个位置参数并且IFS为空格时，"$*"相当于"$1 $2"
	"$@"  会把所有位置参数当成一个单独的字段，如果没有位置参数（$#为0），则"$@"展开为空（不是空字符串，而是空列表），如果存在一个位置参数，则"$@"相当于"$1"，如果有两个参数，则"$@"相当于"$1"  "$2"等等


$?	用于记录上一条命令的执行状态 
		0---255
		0：执行成功 
$$	获取当前执行Shell脚本的进程号（PID）
$!	获取上一个在后台工作的进程的进程号
$_	获取在此之前执行的命令或脚本的最后一个参数


特殊扩展变量
可以man bash 命令，然后搜索"Parameter Expansion"来查找相关的内容帮助

${parameter:-word}       如果parameter的变量值为空或没赋值，则返回word字符串并代替变量的值（变量没定义，返回备用的值，防止变量为空或没定义报错）

${parameter:=word}      如果parameter的变量值为空或没赋值，。。。同上，（变量没定义为防止出错，找的备胎变量）

${parameter:?word}      如果parameter的变量值为空或者没赋值，word字符串就作为标准错误输出，否则出书变量的值（捕捉由于变量未定义导致的错误，并退出）

${parameter:+word}      若果parameter的变量值为空或者未赋值，则什么都不做，否则word字符串将代替变量的值。
```



## 三、判断

```shell
条件判断

-z 判断字符串是否为空串
-n	字符段长度是否非零的 如果结果为真值 返回值为0 如果结果为假值
-eq	等于
-gt	大于
-lt 小于
-ge	大于等于
-le	小于等于
-ne 不等于

根据文件类型判断
-d	文件存在且必须是目录
-e	文件存在，不判断文件类型
-f	文件存在且是标准普通文件
-h	文件存在且是软连接，同 -L
-r	文件存在且可读
-w	文件存在且可写
-x	文件存在且可执行
-s 文件存在且至少有一个字符

双目表达式
单目表达式， 所有的单目表达式都可以使用!表示取反   [ ! -f file_name ]

if标准写法
if 条件; then
   操作语句
   操作语句
   ....
else
   操作语句 
   操作语句 
fi  

[ file1 -ef file2 ]	两个文件有相同的设备编号和inode编号 (判断硬链接)

if [ 'a' != 'b'];then
	echo 'a'
fi

以上可以简写为: [ 'a' != 'b' ] && echo 'a'

```

#### case 语法

```shell
read -p "Enter string: " str_01

case $str_01 in
   linux|Linux)
     echo "CentOS"
     ;;
   windows|Windows)
     echo "Microsoft"
     ;;
   *)
     echo "Other"
     ;;
esac


```

#### for 循环

```shell
# 普通循环
sum=0

for i in `seq 100`; do
    let sum=$sum+$i
done

echo $sum



# 类C写法
for ((i=1;i<8;i++));do
    cmd1
done
```

#### while循环

```shell
while 条件; do
    操作语句 
    操作语句 
    存在一条可以改变条件真假的语句
done 
```

#### until循环

```shell
当条件为假时才循环
until 条件测试 ;do
  cmd1
done
```



#### 数组与循环

```shell

# ca.crt        /etc/openvpn/easy-rsa/pki
# server.crt    /etc/openvpn/easy-rsa/pki/issued
# user.crt      /etc/openvpn/easy-rsa/pki/issued
# user.key      /etc/openvpn/easy-rsa/pki/private
# ca.key        /etc/openvpn/easy-rsa/pki/private
# ta.key        /etc/openvpn/easy-rsa


# 定义用户
accounts=(
client2,用户2
client3,用户3
client4,用户4
)


OPENVPN_DIR="/etc/openvpn"
EASY_RSA_DIR="/etc/openvpn/easy-rsa"

# 分配好的证书
OVPN_DIR="/tmp/openvpn"

# 创建key
# $1 用户证书名  
create_key(){
  [[ ! -f ${EASY_RSA_DIR}/easyrsa ]]  && exit 55
  cd ${EASY_RSA_DIR}/ && ./easyrsa build-client-full $1 nopass
}


# $1 用户名
create_ovpn(){
  [[ ! -f ${OVPN_DIR} ]] && mkdir -p ${OVPN_DIR}
  cat > ${OVPN_DIR}/$1.ovpn << EOF
client
dev tun
proto tcp
remote vpn.local.com 50000
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
comp-lzo
verb 3
EOF

}

# $1 用户名
insert_file(){
  ca=`cat ${EASY_RSA_DIR}/pki/ca.crt`
  user_crt=`cat ${EASY_RSA_DIR}/pki/issued/$1.crt`
  user_key=`cat ${EASY_RSA_DIR}/pki/private/$1.key`
  ta=`cat ${EASY_RSA_DIR}/ta.key`

cat >> ${OVPN_DIR}/$1.ovpn << EOF
<ca> 
${ca}
</ca>
<cert> 
${user_crt}
</cert>
<key> 
${user_key}
</key>
key-direction 1 
<tls-auth> 
${ta}
</tls-auth>
EOF

}

## main
for aobj in ${accounts[@]}
do
  arr=(${aobj//,/ })
  actName=${arr[0]}
  chineseName=${arr[1]}

  create_ovpn ${actName}
  create_key ${actName}
  insert_file ${actName}
done
```



## 四、高级语法

#### 可选参数

```shell
while getopts ":a:b:c:" opt
do
    case $opt in
        a)
        echo "参数a的值$OPTARG"
        ;;
        b)
        echo "参数b的值$OPTARG"
        ;;
        c)
        echo "参数c的值$OPTARG"
        ;;
        ?)
        echo "未知参数"
        exit 1;;
    esac
done


```

### sed

```
d  删除符合条件的行
  # sed '1,2d' /etc/inittab 
  删除文件中包含oot的行
  # sed '/oot/d' /etc/fstab 
  删除第1行及其后2行
  # sed '1,+2d' /etc/fstab 
  删除第1行
  # sed '1d' /etc/fstab 
  删除以/开头的行
  # sed '/^\//d' /etc/fstab 
  
a \string	在符合条件的行后追加新行，string为追加的内容
  在以/开头的行后面追加# hello world 
  # sed '/^\//a \# hello world' /etc/fstab 
  
  在以/开头的行后面追加两行内容，分别为# hello worl  # hello linux 
  # sed '/^\//a \# hello world\n# hello linux' /etc/fstab 
	
i \string	在符合条件的行前添加新行，string为追加的内容
   在文件第1行添加# hello world 
   # sed '1i \# hello world' /etc/fstab 

c \string 	替换指定行的内容
   将文件中最后一行内容替换为End Of File
   # sed '$c \End Of File' /1.txt 
   
   # sed '7c \SELINUX=disabled' /etc/sysconfig/selinux 
```

### awk

```
# awk -F: '/^r/{print $1}' /etc/passwd
```

