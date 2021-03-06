# k8s集群安装
## 一、概述
1. 目前安装版本为官方最新版本1.11.2（截止2018-08-27）。
2. 安装脚本主要参考kubernetes官方文档，采用kubeadm来搭建集群，与官方最大的区别是使用国内镜像源替换了google。
3. 操作系统为CentOS7.5，所有脚本均使用root用户运行，如果在非root用户下，请加上sudo


## 二、环境
| 服务器 | ip | 组件 | 描述 |
| - | - | - | - |
| 虚拟IP | 192.168.56.50 | - | 虚拟IP |
| master-1 | 192.168.56.51 | kubelet,docker | 主master节点 |
| master-2 | 192.168.56.52 | kubelet,docker | 从master节点 |
| node-1 | 192.168.56.53 | kubelet,docker | 工作节点1 |
| node-2 | 192.168.56.54 | kubelet,docker | 工作节点2 |


## 三、安装步骤
### 1. 准备
1. 关闭selinux和防火墙并重启（所有服务器均执行该操作）
```shell
# 关闭selinux
vim /etc/selinux/config

SELINUX=disabled # 修改该值

# 禁用防火墙的开机自启动
systemctl disable firewalld

# 重启
reboot
```

2. master-1可以免密ssh到所有节点（master-1执行）
```shell
# 1. 生成公钥，输入命令后一直Enter
ssh-keygen -t rsa
# 2. 将公钥发布到其它节点
ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.56.52   # 其它节点命令一样，不在一一列举
```

### 2. 安装
```shell
# 1. 下载压缩包到master-1节点，并解压（如果下载的是其它分支或者tag，请注意压缩包和解压后的目录名，不要照搬命令）
cd /root
wget https://github.com/ws1990/k8s-install/archive/master.zip
unzip master.zip
rm -f master.zip
cd k8s-install-master

# 2. 修改配置文件kubernetes.conf
vim kubernetes.conf

hostname=master-1,master-2,node-1,node-2 # 所有的hostname，先master，后node，与master_ip，node_ip的顺序一一对应
master_ip=192.168.56.51,192.168.56.52 	# 所有master节点的ip，第一个为主master节点
node_ip=192.168.56.53,192.168.56.54 	# 所有node节点的ip
load_balancer_dns=192.168.56.50 		# 负载均衡的虚拟IP
load_balancer_port=6443 				# 负载均衡的端口

# 3. 执行install.sh脚本
./install.sh
```


## 四、参考网址
[kubernetes官方安装手册](https://kubernetes.io/docs/setup/independent/high-availability/)
