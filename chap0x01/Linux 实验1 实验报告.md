Linux实验1——实验报告

## 软件环境

Virtualbox

Ubuntu 20.04 server 64bit

## 实验问题

1. 如何配置无人值守安装iso并在Virtualbox中完成自动化安装？
2. 如何使用sftp在虚拟机和宿主机之间传输文件？

## 实验过程

### 手动安装Ubuntu

1. 从课件连接下载安装镜像，挂载后启动虚拟机，出现以下错误![手动安装报错1](img\手动安装报错1.PNG)

   故选择从官网下载安装镜像ubuntu-20.04.2-live-server-amd64.iso，并核对校验和![下载安装镜像](img\下载安装镜像.png)

2. 用Virtualbox新建虚拟机，将安装镜像挂载到虚拟机，并设置双网卡net+host only，启动虚拟机进行配置，手动安装，出现以下情况![手动安装报错2](img\手动安装报错2.PNG)

   关闭虚拟机，重新启动，报错消失，手动安装完成![手动安装完成](img\手动安装完成.PNG)

### 制作无人值守安装镜像

1. 用cmd连接虚拟机

   ```
   ssh akihi@192.168.79.7
   ```

2. 从虚拟机中获取autoinstall-user-data

   ```
   sudo su -  #获取权限
   cat /var/log/installer/autoinstall-user-data #查看文件内容
   exit
   ```

3. 在宿主机上拷贝autoinstall-user-data文件![拷贝autoinstall-user-data文件](img\拷贝autoinstall-user-data文件.PNG)

   ```
   sudo chown akihi：akihi /var/log/installer/autoinstall-user-data #修改文件属主
   scp akhi@192.168.79.7:/var/log/installer/autoinstall-user-data ./ #将文件拷贝到本地
   ```

4. 对照课件中给出的user-data文件对autoinstall-user-data酌情修改后将文件名改为user-data![对照修改user-data](img\对照修改user-data.PNG)

5. 在主机上创建文件meta-data

   ```
   fsutil file createnew meta-data 0
   ```

6. 用sftp命令将user-data和meta-data从宿主机传至虚拟机![用sftp命令将user-data和meta-data从宿主机传至虚拟机](img\用sftp命令将user-data和meta-data从宿主机传至虚拟机.PNG)

   ```
   sftp akihi@192.168.79.7 #连接远程虚拟机
   sftp> put C:\Users\lenovo\meta-data #将meta-data文件传到虚拟机中
   sftp> put C:\Users\lenovo\user-data #将user-data文件传到虚拟机中
   sftp> pwd #查看文件在远程主机的位置
   sftp> exit #退出sftp
   ```

7. cmd中用ssh连接虚拟机，安装genisoimage，制作iso文件![安装genisoimage，制作iso文件](img\安装genisoimage，制作iso文件.PNG)

   ```
   sudo apt install genisoimage #安装genisoimage
   genisoimage -input-charset utf-8 -output init.iso -volid cidata -joliet -rock user-data meta-data #创造镜像文件
   ```

8. 将生成的init.iso文件传至宿主机，并将文件名改为focal-init.iso

   ![init.iso文件传至宿主机](img\init.iso文件传至宿主机.PNG)

### 用配置好的无人值守安装镜像安装虚拟机

1. 新建一个虚拟机![新建一个虚拟机](img\新建一个虚拟机.PNG)

2. 配置双网卡net+host-only，移除控制器IDE，在控制器SATA下先后挂载纯净版Ubuntu安装镜像文件和focal-init.iso，启动虚拟机开始无人值守安装

   ![挂载安装镜像](img\挂载安装镜像.PNG)

## 问题及解决方法

#### 手动安装虚拟机报错

1. ![手动安装报错1](img\手动安装报错1.PNG)安装镜像出错，比对好文件大小并检查校验和后重新下载

2. ![手动安装报错2](img\手动安装报错2.PNG)关闭虚拟机，重新启动后安装成功（没找到原因）

#### cat命令无权限

利用sudo su -命令修改权限

#### scp命令无权限

利用sudo chown命令更改文件属主

## 参考链接

学习sftp操作的相关连接 https://www.cnblogs.com/afeige/p/12144296.html

制作iso文件https://gist.github.com/bitsandbooks/6e73ec61a44d9e17e1c21b3b8a0a9d4c

