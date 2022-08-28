# 权限操作

## 权限

```
drwx------. 5 lz   lz    145 Jul 11 15:21 .
drwxr-xr-x. 3 root root   16 Jul 11 13:34 ..
-rw-------. 1 lz   lz   1131 Jul 11 15:21 .bash_history
-rw-r--r--. 1 lz   lz     18 Apr  1  2020 .bash_logout
-rw-r--r--. 1 lz   lz    193 Apr  1  2020 .bash_profile
-rw-r--r--. 1 lz   lz    231 Apr  1  2020 .bashrc
drwxrwxr-x. 2 lz   lz      6 Jul 11 14:16 document
drwxrwxr-x. 2 lz   lz      6 Jul 11 14:16 download
drwxrwxr-x. 2 lz   lz      6 Jul 11 14:12 public
-rw-------. 1 lz   lz    801 Jul 11 15:21 .viminfo
```

+ **文档类型+权限列表+ 子文件数+文件拥有者+文件所属组+文件大小+最新修改时间+文件名**
+ 文件类型：
  + 当为[ d ]则是目录
  + 当为[ - ]则是文件
  + 若是[ l ]则表示为软连结

## 权限变更

## 默认新建权限

+ 查看默认权限：`umask -S`



### chown ：改变文件拥有者

+ **更改用户权限：**`chown 新用户 目标文件` / `chown 新用户:新组 目标文件`
+ 添加用户：`useradd 用户`
+ 删除用户及home目录：`userdel -r 用户名`
+ 更改密码：`passwd 用户名`
+ 更改用户属组：`usermod -g 用户组 用户名`
+ 将用户添加进某个组：`usermod -a -G 用户组 用户名`



### chgrp ：改变文件所属群组权限

+ **更改组权限：**`chgrp 新用户组 目标文件`
+ 查看更改历程：`chgrp -v 用户组 文件` 查看指定组与文件之间的变更记录
+ 查看用户组：
  + 查看当前用户所在的组：`groups`
  + 查看所有的组：`cat /etc/group`

+ 添加用户组：`groupadd 选项 用户组`

  + -g GID 给用户组分配组ID
  + -og 允许分配已存在的id

+ 删除用户组：`groupdel 用户组`

+ 修改用户组：`groupmod 选项 用户组`

  + -g 新GID 
  + -n 新组名

  

### chmod ：改变文件的权限

+ `chmod -R ug+w,o-rw 文件`
  + a(所有)、u(主)、g(组)、o(访客)
  + +，新增权限
  + ，移除权限
+ `chmod -R 770 文件 `
  + r:4、w:2、x:1



# 文件操作

![image-20220711183313483](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220711183313483.png)



## 创建、复制、移动、删除

### 创建

**文件夹：**

+ `mkdir dir1 dir2 dir3`
+ `mkdir -p  dir1/dir2`

**文件：**

+ `touch 文件`

**软连接：**

+ `ln -s 文件名 连接名`

### 复制

+ `cp a.txt b.txt`，把a的内容复制到b，可用于文件备份
+ `cp a.txt dir1 `，把a拷贝到其他目录
+ `cp -r dir1 dir2`，把目录整体拷贝到其他目录

### 移动，改名

![image-20220711185642886](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220711185642886.png)

### 删除

+ `rm a.txt`，删除文件
+ `rm -rf dir1`，删除文件夹
+ `rmdir` ，删除空文件夹



## 查看，更改

**查看文件类型：**

+ `file 文件名`

**查看文件内容：**

+ `cat -n 小文件`，附带行号
+ `less -N 大文件`，分页查看
+ `head -n 100 文件名`，查看文件首100行
+ `tail -n 100 file_name`，查看文件尾100行
+ `tail -f 20 info.log `，滚动查看最后20行

**查看文件分布情况：**

+ `ls -R`

**更改内容：**

+ `echo 文本 > 文件名`，覆盖写
+ `echo 文本 >> 文件名`，追加写



## 查找 

+ **功能文件：**`which 方法关键字`
+ **查找系统特定目录：**`whereis 文件`
+ **最终查找，慢：**`find 路径 参数`





## 解压缩

**.gz（压缩文件）**/ **.tar（文件包）**/ **.tar.gz（压缩文件包）**

### .gz

+ 压缩：`gzip 文件`
+ 解压：`gunzip 文件`

### .tar

+ 只打包文件：`tar -cvf 文件包名 目标目录1 目标目录2`
+ 追加文件到文件包：`tar -rvf 文件包名 目标目录`
+ 展示文件包下的文件列表：`tar -tvf test.tar`
+ 只解包：`tar -xpvf 文件包 -C 目标文件夹 `，必须是存在的文件夹

### .tar.gz

+ 打包加压缩：`tar -cvzf 压缩文件名 目标目录`
+ 解包加解压：`tar -xzvf test.tar.gz -C 目标目录`
+ 分割大压缩包：`split -b 6M 大压缩包 -d 小压缩包命名前缀 `
+ 合并小压缩包：`cat 小压缩包命名前缀* > 新压缩包名 `

# 进程操作

## 查看进程

+ 查看所有进程详情：`ps aux`

  ![image-20220713180802772](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220713180802772.png)

  + %MEM：物理内存占用率
  + VSZ：虚拟内存量
  + RSS：固定的内存量
  + TTY：运作的终端机。? 与终端无关、 tty1-tty6 是本机、 pts/n 是远程
  + STAT：当前状态
  + START：启动时间
  + TIME：进程使用CPU的累计时间
  + COMMAND：触发指令

+ 查看用户所属进程：`ps -l`

  ![image-20220713180930736](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220713180930736.png)

  + F：指代权限
  + S：指代进程状态。R[running], S[sleep], T[stop], Z[Zombie], D[阻塞]
  + UID/PID/PPID：用户ID、进程ID、父进程ID
  + C：CPU占用率
  + PRI / NI：优先级
  + ADDR：内存位置
  + SZ：内存占用量
  + WCHAN：运行状态。"-"代表正在运行
  + CMD：触发指令

+ 查看进程关联情况：`pstree -A`



## 进程终止

+ `kill-9  pid`



# 高效Shell指令

+ 批量修改文件：`sed 's/6401/6408/g' redis-6401.conf > redis-6408.conf`
+ 批量删除redis进程：` ps -ef | grep redis | grep -v grep | cut -c 9-15 | xargs kill -9`



















