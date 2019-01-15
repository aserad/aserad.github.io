---
title: FastDFS的安装与使用
category: FastDFS
tag:
    - fastdfs
    - nginx
---

## FastDFS的介绍
FastDFS只有两个角色：Tracker server和Storage server。Tracker server作为中心结点，其主要作用是负载均衡和调度。Tracker server在内存中记录分组和Storage server的状态等信息，不记录文件索引信息，占用的内存量很少。另外，客户端（应用）和Storage server访问Tracker server时，Tracker server扫描内存中的分组和Storage server信息，然后给出应答。
  
Storage server直接利用OS的文件系统存储文件。FastDFS不会对文件进行分块存储，客户端上传的文件和Storage server上的文件一一对应。

在FastDFS中，客户端上传文件时，文件ID不是由客户端指定，而是由Storage server生成后返回给客户端的。文件ID中包含了组名、文件相对路径和文件名，Storage server可以根据文件ID直接定位到文件。

## FastDFS的安装与配置（CentOS 7 64位系统）
- 安装依赖包
    - 执行 `git clone https://github.com/happyfish100/libfastcommon.git` 命令将依赖包下载到本地，若执行该命令出现bash: git: command not found...错误说明系统上没有安装git,需先安装git，安装命令为`yum install git`，安装完成再下载
    - 下载完后cd到libfastcommon目录， 分别执行`./make.sh`和`./make.sh install`这两个命令，如果执行第一个命令出现如下错误
```
./make.sh: line 14: gcc: command not found
./make.sh: line 15: ./a.out: No such file or directory
cc -Wall -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -g -O3 -c -o hash.o hash.c
make: cc: Command not found
make: *** [hash.o] Error 127
```
    需要先安装gcc编译器，安装命令为 `yum install -y gcc` 

- 安装 FastDFS
    - 执行`git clone https://github.com/happyfish100/fastdfs.git`命令下载FastDFS的安装包
    - 下载完成后cd到fastdfs目录， 分别执行`./make.sh`和`./make.sh install`命令

- 配置跟踪服务器 tracker
    - 执行 `cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf`命令拷贝一份配置文件
    - 在/root/Documents目录中创建目录 fastdfs/tracker 来存储数据和日志等信息, 创建命令为 `mkdir -p /root/Documents/fastdfs/tracker`
    - 编辑 /etc/fdfs/tracker.conf 配置文件， 修改其中的 base_path=/root/Documents/fastdfs/tracker

- 配置存储服务器 storage
    - 执行 `cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf`命令拷贝一份配置文件
    - 在/root/Documents目录中创建目录 fastdfs/storage 来存储数据和日志等信息, 创建命令为 `mkdir -p /root/Documents/fastdfs/storage`
    - 编辑 /etc/fdfs/storage.conf 配置文件， 修改以下内容
```
base_path=/root/Documents/fastdfs/storage
store_path0=/root/Documents/fastdfs/storage
tracker_server=本机的ip:22122
```
    其中base_path是存储服务器的数据和日志的存储路径，store_path0是上传的文件的存储路径，tracker_server是跟踪服务器的地址

- 启动 tracker 和 storage
    - `service fdfs_trackerd start`
    - `service fdfs_storaged start`

- 测试是否安装成功
    - 配置客户端的配置文件
        - 执行 `cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf`命令拷贝一份配置文件
        - 编辑 /etc/fdfs/client.conf 文件，修改以下内容
```
base_path=/root/Documents/fastdfs/tracker
tracker_server==本机的ip:22122
```
    - 上传文件测试，执行命令 `fdfs_upload_file /etc/fdfs/client.conf 要上传的图片的路径`，如果返回类似 group1/M00/00/00/wKgAaFw7LvaAe0WSAABC_zNYbBE813.jpg 的文件id则说明文件上传成功

## Nginx配合FastDFS的使用（需要安装Nginx和对应的模块 fastdfs-nginx-module）
- 首先下载需要的模块，使用命令 `git clone https://github.com/happyfish100/fastdfs-nginx-module`下载，注意，在哪个目录下执行该命令文件夹就会下载到哪个目录，这里下载到/root目录下了
- Nignx的下载与安装
    - 去[Nginx官网](http://nginx.org/en/download.html)下载压缩包，这里下载的是`nginx-1.15.8.tar.gz`
    - 安装依赖包
```
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```
    - 使用命令 `tar -zxvf nginx-1.15.8.tar.gz -C /usr/local/` 将压缩包解压到/user/local目录下
    - 进行configure配置，首先进入解压后的文件夹，再执行命令 `./configure --prefix=/usr/local/nginx --add-module=fastdfs-nginx-module目录的绝对路径/src`，
    - 分别执行`make`和`make install`命令
- 配置 fastdfs-nginx-module 模块
    - 把 fastdfs-nginx-module文件夹下的src下的 mod_fastdfs.conf 文件拷贝到 /etc/fdfs/mod_fastdfs.conf
    因为我的fastdfs-nginx-module文件夹在root目录下，所以我执行的命令为`cp /root/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/mod_fastdfs.conf`
    - 修改 /etc/fdfs/mod_fastdfs.conf 文件， 要修改的内容如下
```
connect_timeout=10
tracker_server=本机的ip:22122
url_have_group_name=true
store_path0=/root/Documents/fastdfs/storage  # 该目录可以自定义
```
- 配置FastDFS
    - 拷贝fastdfs目录中的conf目录下的http.conf文件和mime.types文件
    `cp /root/fastdfs/conf/http.conf /etc/fdfs/http.conf`
    `cp /root/fastdfs/conf/mime.types /etc/fdfs/mime.types`
- 修改Nginx配置文件，编辑 /usr/local/nginx/conf/nginx.conf 文件，加入以下内容
```
server {
	listen       8888;
	server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}

```
- 启动Nginx服务， 进入nginx安装的目录，这里是/usr/local/nginx，然后执行`cd sbin`进入sbin目录，可以看到有一个nginx执行文件，执行`./nginx`命令就可以启动
如果nginx服务已经启动，可以通过`./nginx -s stop`命令来停止服务，使用`ps -ef | grep nginx`可以查看nginx服务是否已经启动
    

- 在进行上述步骤后，访问localhost:8888/group1/M00/00/00/wKgAaFw7LvaAe0WSAABC_zNYbBE813.jpg类似的网址，页面会展示上传的图片，如果报404错误，可以修改/usr/local/nginx/conf/nginx.conf 文件， 加上`user root`，参考博客https://blog.csdn.net/mfanoffice2012/article/details/84976470
