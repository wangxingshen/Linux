# Linux 常用命令速查

> 整理日期：2026-09-14
> 说明：`[]` 表示可选参数，`<>` 表示必填参数

---

## 目录

- [1. 文件与目录](#1-文件与目录)
- [2. 查看文件内容](#2-查看文件内容)
- [3. 权限与属主](#3-权限与属主)
- [4. 查找与搜索](#4-查找与搜索)
- [5. 文本处理（grep/sed/awk）](#5-文本处理grepsedawk)
- [6. 重定向与管道](#6-重定向与管道)
- [7. 进程与系统监控](#7-进程与系统监控)
- [8. 磁盘与内存](#8-磁盘与内存)
- [9. 压缩与打包](#9-压缩与打包)
- [10. 网络](#10-网络)
- [11. 用户与权限提升](#11-用户与权限提升)
- [12. systemd 服务管理](#12-systemd-服务管理)
- [13. 远程与文件传输](#13-远程与文件传输)
- [14. 软件包管理](#14-软件包管理)
- [15. 环境变量与 Shell](#15-环境变量与-shell)
- [16. 高频一行流](#16-高频一行流)
- [17. 附：机器人 / ROS 2 常用](#17-附机器人--ros-2-常用)

---

## 1. 文件与目录

```bash
pwd                      # 显示当前路径
ls -alh                  # 列出全部文件，含隐藏，人类可读大小
ls -lt                   # 按修改时间排序（新→旧）
ls -ltr                  # 按修改时间倒序（旧→新，看最新改动好用）
cd /path                 # 切换目录
cd -                     # 回到上一个目录
cd ~                     # 回 home
tree -L 2                # 树状显示（深度 2），需 apt install tree

mkdir -p a/b/c           # 递归创建目录
touch file.txt           # 创建空文件 / 更新时间戳
cp -r src/ dst/          # 递归复制
cp -a src/ dst/          # 保留权限时间戳递归复制（归档式）
mv old new               # 移动 / 重命名
rm file                  # 删文件
rm -rf dir/              # 递归强制删目录（危险，确认路径）
rmdir dir                # 删空目录

ln -s /target link       # 创建软链接
readlink -f link         # 解析软链接真实路径
realpath file            # 输出规范绝对路径
stat file                # 详细元信息（大小/权限/时间）
du -sh dir/              # 目录总大小
du -sh * | sort -h       # 列出各子项大小并排序
df -h                    # 各分区使用情况
```

**注意**：`rm -rf` 不可恢复，执行前用 `pwd` 和 `ls` 确认位置。

---

## 2. 查看文件内容

```bash
cat file                 # 全量输出
cat -n file              # 带行号
less file                # 分页查看（/ 搜索，q 退出，G 末尾，g 开头）
head -n 20 file          # 前 20 行
tail -n 20 file          # 后 20 行
tail -f log.txt          # 实时跟踪日志（Ctrl+C 退出）
tail -F log.txt          # 跟踪并处理日志轮转（推荐）
wc -l file               # 统计行数
file file                # 判断文件类型
```

---

## 3. 权限与属主

权限位：`r=4 w=2 x=1`，三段分别是 **属主 / 属组 / 其他**。

```bash
chmod 755 file           # rwxr-xr-x
chmod 644 file           # rw-r--r--
chmod +x script.sh       # 加执行权限
chmod -R 755 dir/        # 递归改权限

chown user:group file    # 改属主和属组
chown -R user dir/       # 递归改属主
chgrp group file         # 只改属组

umask                    # 查看默认权限掩码
```

常见数值：

| 数值 | 权限 | 典型用途 |
|---|---|---|
| 644 | rw-r--r-- | 普通文件 |
| 755 | rwxr-xr-x | 可执行 / 目录 |
| 600 | rw------- | 私钥、密钥文件 |
| 700 | rwx------ | 私有目录 |

---

## 4. 查找与搜索

```bash
find /path -name "*.log"                 # 按文件名查
find /path -iname "*.LOG"                # 忽略大小写
find . -type f -name "*.py"              # 只找普通文件
find . -type d -name "build"             # 只找目录
find . -size +100M                       # 大于 100M 的文件
find . -mtime -7                         # 7 天内修改过
find . -name "*.tmp" -delete             # 查并删除
find . -name "*.sh" -exec chmod +x {} \; # 查并执行命令

which python3                            # 命令所在路径
whereis python3                          # 二进制/手册/源码位置
type ls                                  # 判断是别名/内建/外部命令
locate file                              # 走数据库，快（需 updatedb）
```

---

## 5. 文本处理（grep/sed/awk）

```bash
# grep：搜索文本
grep "error" log.txt                     # 基础搜索
grep -i "error" log.txt                  # 忽略大小写
grep -n "error" log.txt                  # 显示行号
grep -r "TODO" src/                      # 递归搜索目录
grep -rn --include="*.py" "def main" .   # 指定文件类型递归
grep -v "debug" log.txt                  # 反向匹配（排除）
grep -c "error" log.txt                  # 只输出匹配行数
grep -E "err(or|ors)" log.txt            # 扩展正则

# sed：流编辑
sed -n '10,20p' file                     # 打印 10-20 行
sed 's/old/new/' file                    # 替换每行第一个
sed 's/old/new/g' file                   # 替换所有
sed -i 's/old/new/g' file                # 直接改文件（先备份！）
sed -i.bak 's/old/new/g' file            # 改文件并留 .bak
sed '/^#/d' file                         # 删掉注释行
sed -i '3d' file                         # 删第 3 行

# awk：按列处理
awk '{print $1, $3}' file                # 打印第 1、3 列
awk -F: '{print $1}' /etc/passwd         # 指定分隔符
awk '$3 > 100 {print $0}' file           # 条件过滤
awk '{sum += $1} END {print sum}' file   # 求和
```

---

## 6. 重定向与管道

```bash
cmd > file               # 覆盖写入 stdout
cmd >> file              # 追加
cmd 2> err.log           # stderr 重定向
cmd > out.log 2>&1       # stdout+stderr 一起写
cmd &> out.log           # 同上（bash 简写）
cmd > /dev/null 2>&1     # 丢弃全部输出
cmd < input.txt          # stdin 重定向
cmd1 | cmd2              # 管道
cmd1 | tee out.log       # 同时输出到屏幕和文件
yes | rm -i *.tmp        # 自动回答 y
```

---

## 7. 进程与系统监控

```bash
ps aux                   # 全量进程
ps -ef | grep python     # 找特定进程
pgrep -a python          # 按名字找并显示命令行
pidof nginx              # 按名字取 PID

top                      # 实时监控（M 按内存排，P 按 CPU 排，q 退出）
htop                     # 更友好（需安装）
kill <PID>               # 发 SIGTERM 优雅结束
kill -9 <PID>            # 强杀（发给 SIGKILL）
pkill -f "python app.py" # 按命令行模式杀
killall nginx            # 按进程名杀

nohup cmd &              # 后台运行，忽略挂断
jobs                     # 查看当前 shell 后台任务
fg %1 / bg %1            # 前台/后台切换任务
Ctrl+Z                   # 挂起当前进程
Ctrl+C                   # 中断当前进程

uptime                   # 运行时长 + 负载
uname -a                 # 内核与架构
lscpu                    # CPU 信息
lsblk                    # 块设备/分区树
dmesg | tail             # 内核日志（看 USB/驱动问题）
journalctl -xe           # 系统日志
```

---

## 8. 磁盘与内存

```bash
df -h                    # 分区使用率
df -i                    # inode 使用率（空间够但写不进时看这个）
du -sh *                 # 当前目录各项大小
du -h --max-depth=1 .    # 一级子目录大小
du -sh * | sort -rh | head -10   # Top 10 大文件/目录

free -h                  # 内存（含 swap）
vmstat 1                 # 每秒采样系统状态
iostat -x 1              # 磁盘 IO
sync                     # 刷缓存到磁盘
echo 3 > /proc/sys/vm/drop_caches  # 清缓存（需 root）
```

---

## 9. 压缩与打包

```bash
tar -czvf out.tar.gz dir/          # 打包 gzip
tar -xzvf out.tar.gz               # 解包 gzip
tar -xzvf out.tar.gz -C /target    # 解到指定目录
tar -tzvf out.tar.gz               # 只看内容不解包
tar -cjvf out.tar.bz2 dir/         # bzip2
tar -cJvf out.tar.xz dir/          # xz（压缩率高，慢）

zip -r out.zip dir/                # zip 压缩
unzip out.zip                      # 解压
unzip -l out.zip                   # 列举内容
unzip out.zip -d /target           # 解到指定目录

gzip file / gunzip file.gz         # 单文件压缩
xz -9 file                         # 高压缩
```

**记忆**：`tar` 参数 `c`=创建 `x`=解压 `t`=列表 `z`=gzip `v`=详细 `f`=文件名。

---

## 10. 网络

```bash
ip addr / ip a                    # 查看 IP（替代 ifconfig）
ip route / ip r                   # 路由表
ip link set eth0 up/down          # 启停网卡

ping -c 4 8.8.8.8                 # 测连通
ping -I eth0 8.8.8.8              # 指定网卡 ping（多网卡场景）
curl -I https://example.com       # 只看响应头
curl -O https://x.com/file.tar.gz # 下载并保留原名
curl -L -o out.zip URL            # 跟随重定向下载
wget URL                          # 下载
wget -c URL                       # 断点续传

ss -tulnp                         # 查看监听端口（替代 netstat）
ss -tnp | grep 8080               # 查哪个进程占用端口
lsof -i :4000                     # 端口占用（需安装 lsof）
nslookup domain / dig domain      # DNS 解析
traceroute domain                 # 路由追踪
nc -zv host 22                    # 测端口通不通
```

---

## 11. 用户与权限提升

```bash
whoami                           # 当前用户
id                               # UID/GID/所属组
who / w                          # 登录用户
su - user                        # 切换用户（加载其环境）
sudo cmd                         # 以 root 执行
sudo -i                          # 切到 root 交互 shell
sudo -u user cmd                 # 以指定用户执行

useradd -m -s /bin/bash tom      # 建用户 + home + shell
passwd tom                       # 设密码
usermod -aG sudo tom             # 加入 sudo 组
userdel -r tom                   # 删用户及 home

groups                           # 当前用户所属组
```

---

## 12. systemd 服务管理

```bash
systemctl status nginx           # 查看状态
systemctl start/stop/restart nginx
systemctl enable nginx           # 开机自启
systemctl disable nginx          # 取消自启
systemctl is-enabled nginx
systemctl list-units --type=service --state=running

journalctl -u nginx -f           # 跟服务日志
journalctl -u nginx --since "1 hour ago"
journalctl -b -p err             # 本次启动的错误日志

# 默认启动目标（图形 ↔ 多用户）
systemctl get-default
systemctl set-default multi-user.target   # 关闭图形界面
systemctl set-default graphical.target
```

---

## 13. 远程与文件传输

```bash
ssh user@host                    # 登录
ssh -p 2222 user@host            # 指定端口
ssh -i ~/.ssh/id_rsa user@host   # 指定私钥
ssh user@host "uptime"           # 远程执行单条命令

ssh-keygen -t ed25519            # 生成密钥
ssh-copy-id user@host            # 上传公钥免密登录

scp file user@host:/path/        # 上传
scp user@host:/path/file .       # 下载
scp -r dir/ user@host:/path/     # 递归

rsync -avz --progress src/ user@host:/dst/   # 增量同步（推荐）
rsync -avz --delete src/ user@host:/dst/     # 目标端多余文件删除（危险）
```

**SSH 免密脚本化**（Windows Git Bash 下无 tty 时用）：

```bash
SSH_ASKPASS=/path/askpass.sh SSH_ASKPASS_REQUIRE=force ssh user@host
```

多网卡同时存在时，指定源网卡：

```bash
ssh -o BindAddress=192.168.168.50 robot@192.168.168.100
```

---

## 14. 软件包管理

```bash
# Debian / Ubuntu
sudo apt update                  # 更新索引
sudo apt upgrade                 # 升级已装包
sudo apt install pkg             # 安装
sudo apt remove pkg              # 卸载
sudo apt purge pkg               # 卸载 + 清配置
apt search keyword               # 搜索
apt list --installed             # 已装列表
dpkg -l | grep pkg               # 查是否已装
sudo dpkg -i pkg.deb             # 装本地 deb
sudo apt -f install              # 修复依赖

# RHEL / CentOS / Fedora
sudo yum install pkg / sudo dnf install pkg
rpm -qa | grep pkg

# Arch
sudo pacman -S pkg / sudo pacman -Rns pkg
```

---

## 15. 环境变量与 Shell

```bash
echo $PATH                       # 查看变量
export VAR=value                 # 当前 shell 生效
env                              # 列出所有环境变量
printenv VAR
unset VAR                        # 删除变量

# 持久化（~/.bashrc 用于交互 shell，~/.profile 用于登录）
echo 'export VAR=value' >> ~/.bashrc
source ~/.bashrc                 # 使其立即生效

alias ll='ls -alh'               # 别名
alias                            # 查看所有别名
unalias ll

history                          # 命令历史
history | grep ssh
Ctrl+R                           # 反向搜索历史（最好用的一招）
!!                               # 重复上一条命令
sudo !!                          # 用 sudo 重复上一条
```

---

## 16. 高频一行流

```bash
# 找最大的 10 个文件
du -ah . | sort -rh | head -10

# 统计当前目录文件数
ls -1 | wc -l

# 批量重命名后缀
for f in *.txt; do mv "$f" "${f%.txt}.md"; done

# 杀掉占用 8080 端口的进程
lsof -ti:8080 | xargs kill -9

# 查某 IP 的所有连接
ss -tnp | grep 192.168.3.50

# 监控日志并只筛错误
tail -f app.log | grep --line-buffered -i error

# 每秒刷新看内存占比
watch -n 1 'free -h'

# 替换目录下所有文件的字符串
grep -rl "old_str" . | xargs sed -i 's/old_str/new_str/g'

# 确认两文件差异
diff -u a.txt b.txt

# 网络多网卡时强制走某网卡
curl --interface eth0 https://ip.sb
```

---

## 17. 附：机器人 / ROS 2 常用

```bash
# ROS 2 环境
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 topic list
ros2 topic echo /cmd_vel
ros2 topic hz /scan                # 看话题频率
ros2 node list
ros2 node info /node_name
ros2 param list /node_name
ros2 interface show geometry_msgs/msg/Twist

# 发布/查看
ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.2}, angular: {z: 0.0}}"

# 建包编译
ros2 pkg create --build-type ament_cmake my_pkg
colcon build --symlink-install --packages-select my_pkg

# rosbag
ros2 bag record -o mybag /scan /odom
ros2 bag info mybag
ros2 bag play mybag

# 网络隔离调试（ROS 域）
export ROS_DOMAIN_ID=42
export RMW_IMPLEMENTATION=rmw_zenoh_cpp
export ZENOH_SESSION_CONFIG_URI=/path/to/session.json5   # 注意不是 ZENOH_CONFIG_PATH
```

其他实用工具：

```bash
# 串口调试
ls /dev/ttyUSB* /dev/ttyACM*
sudo chmod 666 /dev/ttyUSB0
minicom -D /dev/ttyUSB0 -b 115200
sudo screen /dev/ttyUSB0 115200    # Ctrl+A K 退出

# 查看 USB 设备
lsusb
dmesg | grep -i usb | tail -20
```

---

## 危险命令黑名单

执行前务必二次确认路径：

| 命令 | 风险 |
|---|---|
| `rm -rf /` 或 `rm -rf /*` | 清空系统 |
| `rm -rf ~` | 清空 home |
| `> file` | 立即清空文件内容 |
| `dd if=... of=/dev/sdX` | 覆盖磁盘，不可逆 |
| `chmod -R 777 /` | 权限全面失控 |
| `mkfs.* /dev/sdX` | 格式化磁盘 |
| `:(){ :|:& };:` | fork 炸弹 |

**习惯**：危险命令先用 `echo` 或 `ls` 走一遍，确认无误再去掉 echo。
