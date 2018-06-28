---
title: 在Ubuntu上安装和配置ftp服务器
categories:
    - linux
    - ftp
tags: 
    - ftp
    - ubuntu
---

FTP是一个相对古老，用于两台电脑之间上传，下载文件的网络协议。然而，FTP的原始安全策略在传输数据，帐号，密码时并没有加密处理。


## 在`Ubuntu`上安装`VsFTP`
> Ubuntu version: Ubuntu 16.04.1 LTS

1. 首先需要安装vsftpd
    1. `sudo apt-get update`
    2. `sudo apt-get install vsftpd`

2. 安装完成，服务器并没有自动开启，因此，我们需要手动开机，并添加到开机启动中
    1. `sudo systemctl start vsftpd`
    2. `sudo systemctl enable vsftpd`

3. 开启`ufw firewall`的20和21接口
    1. `sudo ufw allow 20/tcp`
    2. `sudo ufw allow 21/tcp`
    3. `sudo ufw status`

## 配置和加固`VsFTP`

1. 修改配置文件
    1. `sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.orig`
    2. `sudo vi /etc/vsftpd.conf`
    3. 修改配置文件
    ```
    anonymous_enable=NO             # disable  anonymous login
    local_enable=YES		# permit local logins
    write_enable=YES		# enable FTP commands which change the filesystem
    local_umask=022		        # value of umask for file creation for local users
    dirmessage_enable=YES	        # enable showing of messages when users first enter a new directory
    xferlog_enable=YES		# a log file will be maintained detailing uploads and downloads
    connect_from_port_20=YES        # use port 20 (ftp-data) on the server machine for PORT style connections
    xferlog_std_format=YES          # keep standard log file format
    listen=NO   			# prevent vsftpd from running in standalone mode
    listen_ipv6=YES		        # vsftpd will listen on an IPv6 socket instead of an IPv4 one
    pam_service_name=vsftpd         # name of the PAM service vsftpd will use
    userlist_enable=YES  	        # enable vsftpd to load a list of usernames
    tcp_wrappers=YES  		# turn on tcp wrappers
    ```

2. 当`userlist_deny=YES`并且`userlist_enable=YES`时，位于用户列表的用户是被拒绝访问的，但如果`userlist_deny=NO`，则变成只有位于用户列表下的用户才会允许访问
    ```
    userlist_enable=YES                   # vsftpd will load a list of usernames, from the filename given by userlist_file
    userlist_file=/etc/vsftpd.userlist    # stores usernames.
    userlist_deny=NO 
    ```

3. 配置选项`chroot_local_user=YES`当用户登录的时候，会定位于用户根目录下，并且 VSFTPD 初始是不允许在用户根目录下写入的，需要打开配置`allow_writeable_chroot=YES`
    ```
    chroot_local_user=YES
    allow_writeable_chroot=YES
    ```

4. 重启 `sudo systemctl restart vsftpd`

## 配置 FTP 用户的根目录
1. 创建FTP用户
    1. `sudo useradd -m -c "ftp test user" -s /bin/bash ftptest`
    2. `sudo passwd ftptest`
    3. `echo "ftptest" | sudo tee -a /etc/vsftpd.userlist`
    4. `cat /etc/vsftpd.userlist`

2. 修改配置文件，不予许在根目录写入， `#allow_writeable_chroot=YES`

3. 在用户根目录下创建一个目录，不允许其他用户写入
    ```
    sudo mkdir /home/ftptest/ftp
    sudo chown nobody:nogroup /home/ftptest/ftp
    sudo chmod a-w /home/ftptest/ftp
    ```

4. 创建一个目录用于写入
    ```
    sudo mkdir /home/ftptest/ftp/files
    sudo chown -R ftptest:ftptest /home/ftptest/ftp/files
    sudo chmod -R 0770 /home/ftptest/ftp/files/
    ```

5. 修改配置文件，重新定义用户的根目录位置
    ```
    user_sub_token=$USER          # inserts the username in the local root directory 
    local_root=/home/$USER/ftp    # defines any users local root directory
    ```

6. 重启，`sudo systemctl restart vsftpd`
