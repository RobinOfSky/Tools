####在docker下安装pmm(数据库的监控工具)
**安装docker**
 - 安装依赖包
 ```sudo yum install -y yum-utils device-mapper-persistent-data lvm2```
 - 设置阿里云镜像源
```sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo ```

- 安装docker-ce
    ``` sudo yum install docker-ce```
- 启动docker-ce
  ``` sudo systemctl enable docker
     sudo systemctl start docker
     ```
-  为docker建立用户组（可选）
>docker 命令与 Docker 引擎通讯之间通过 UnixSocket ，但是能够有权限访问 UnixSocket 的用户只有 root 和 docker 用户组的用户才能够进行访问，所以我们需要建立一个 docker 用户组，并且将需要访问 docker 的用户添加到这一个用户组当中来。

- 建立docker用户组
 ``` sudo groupadd docker```
- 添加当前用户到docker组
   ```sudo usermod -aG docker $USER```

 **镜像加速设置**
   ``` sudo mkdir -p /etc/docker ```
   
   ```sudo tee /etc/docker/daemon.json <<-'EOF'{ "registry-mirrors": ["你的加速器地址"]}EOF```

 **重新加载配置，并重启docker服务**
 ``` sudo systemctl daemon-reload   sudo systemctl restart docker  ```
 
**下载容器镜像**
``` docker pull percona/pmm-server:latest```

**建立数据卷容器**
    ```docker create -v /opt/prometheus/data -v /opt/consul-data -v /var/lib/mysql -v /var/lib/grafana --name pmm-data percona/pmm-server:latest /bin/true ```
    
**在建立数据卷容器的时候发生的命名冲突问题，该如何解决？**

_第一种方法就是_
 ```docker kill fd3c0c622af  或者docker rm fd3c0c622af6 ```

_第二种就是_

  ``` docker kill $(docker ps -q); docker rm $(docker ps -a -q) ```
  
 **创建和运行**
   ``` docker run -d -p 80:80 --volumes-from pmm-data --name    pmm-server --restart always percona/pmm-server:latest```
   
 **安装pmm-client**
 
1.下载执行文件

 ``` wget https://www.percona.com/downloads/pmm-client/pmm-client-1.1.1/binary/tarball/pmm-client-1.1.1.tar.gz```
2.解压可执行文件

 ```  tar -zxvf pmm-client-1.1.1.tar.gz```
3.进入解压后的文件

 ```cd pmm-client-1.1.1 ```
4.执行install文件

 ``` ./install```
**下载percona-toolkit**

1.下载和安装percona toolkit的包

 ``` yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm```
2.查看可以安装的包

 ``` yum list | grep percona-toolkit```
3.安装percona toolkit

 ``` yum install percona-toolkit```
出现complete即表示安装成功
4.安装成功后可以通过下列命令确认是否安装成功

 ```pt-query-digest --help    pt-table-checksum --help```
 
**客户端连接server**

 ```pmm-admin config --server 192.168.146.137```
 
 **增加pmm-client监控账号**
 
  ``` GRANT ALL PRIVILEGES ON  *.* TO 'robin'@'192.168.146.137' IDENTIFIED BY 'robin';```
  
**增加pmm客户端监控mysql服务器**

``` pmm-admin add mysql --user robin--password robin--host 192.168.146.137 --port 3306 ``` 
**查看pmm的admin**

 ``` pmm-admin list ```
 
 **常用pmm-client命令介绍**
 
 >添加监控服务
 pmm-admin add                             
检查PMM客户端和PMM服务器之间的网络连接。
pmm-admin check-network                   
配置PMM Client如何与PMM服务器通信。     
pmm-admin config                          
 打印任何命令和退出的帮助                
pmm-admin help                            
打印有关PMM客户端的信息                 
pmm-admin info                            
出为此PMM客户端添加的所有监控服务       
pmm-admin list                            
检查PMM服务器是否存活                   
pmm-admin ping                           
检查PMM服务器是否存活。                 
pmm-admin purge                           
清除PMM服务器上的度量数据               
pmm-admin remove, pmm-admin rm            
删除监控服务                           
pmm-admin repair                          
重启pmm                                 
pmm-admin restart                         
打印PMM Client使用的密码                
pmm-admin show-passwords                
 开启监控服务                            
pmm-admin start                           
停止监控服务                            
pmm-admin stop                            
在卸载之前清理PMM Client                
pmm-admin uninstall         
