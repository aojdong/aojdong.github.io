## linux 配置源

基于Debian的Linux系统使用的是 apt 软件包管理器, 经常会需要配置源, 可以根据需要添加或删除。

- apt 源列表文件 `/etc/apt/sources.list`
    * `deb`：表示**二进制软件包**的下载地址。
    * `deb-src`：表示**源代码软件包**下载地址。

- yum 源配置目录 `/etc/yum.repos.d/`


### 常用的 apt 源：
1. 官方源：http://archive.ubuntu.com/ubuntu/
2. 清华源：https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
3. 阿里云源：http://mirrors.aliyun.com/ubuntu/
4. 中科大源：https://mirrors.ustc.edu.cn/ubuntu/
5. 华为源：https://mirrors.huaweicloud.com/ubuntu/
6. 搜狐源：http://mirrors.sohu.com/ubuntu/
7. 163源：http://mirrors.163.com/ubuntu/
8. 网易源：http://mirrors.163.com/ubuntu/
9. 腾讯源：https://mirrors.cloud.tencent.com/ubuntu/
10. 中国科技大学源：https://mirrors.ustc.edu.cn/ubuntu/

### 常用的yum源：
1. CentOS官方yum源：http://mirror.centos.org/centos/
2. 阿里云yum源：http://mirrors.aliyun.com/centos/
3. 清华大学TUNA镜像站：https://mirrors.tuna.tsinghua.edu.cn/centos/
4. 网易yum源：http://mirrors.163.com/centos/
5. 搜狐yum源：http://mirrors.sohu.com/centos/
6. 华为云yum源：https://mirrors.huaweicloud.com/centos/
7. 中国科技大学yum源：http://mirrors.ustc.edu.cn/centos/

不同的源可能有不同的速度和稳定性，建议按照自己的网络情况选择合适的源。

### 常用的镜像站：
1. 清华大学开源软件镜像站：https://mirrors.tuna.tsinghua.edu.cn/
2. 阿里云开源镜像站：https://mirrors.aliyun.com/
3. 中科大开源软件镜像站：https://mirrors.ustc.edu.cn/
4. 华为开源镜像站：https://mirrors.huaweicloud.com/
5. 搜狐开源镜像站：https://mirrors.sohu.com/
6. 网易开源镜像站：https://mirrors.163.com/
7. 腾讯云开源镜像站：https://mirrors.cloud.tencent.com/
8. Debian官方镜像站：https://www.debian.org/mirror/list
9. Ubuntu官方镜像站：https://mirrors.ubuntu.com/
10. CentOS官方镜像站：https://www.centos.org/download/mirrors/
