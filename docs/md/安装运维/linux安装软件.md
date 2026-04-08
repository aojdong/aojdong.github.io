## linux安装软件

使用包管理器 `apt` 示例
配置好源之后，进行更新， `apt update`

安装软件 `apt install <package-name>`

卸载软件 `apt remove <package-name>`

查看软件版本 `dpkg -l | grep <package-name>`

查看软件安装路径 `dpkg -L <package-name>`


### 安装 git

```shell
apt install git
git --version
```

## 安装 python 3.10.5
Ubuntu 不会提供任意小版本, 所以需要手动编译安装
1. 先安装编译依赖
```shell
apt update
apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev
```

2. 下载 python 3.10.5 源码
```shell
cd /tmp
wget https://www.python.org/ftp/python/3.10.5/Python-3.10.5.tgz
```

3. 解压并编译安装
```shell
tar -xf Python-3.10.5.tgz
cd Python-3.10.5
./configure --enable-optimizations
make -j$(nproc)
make altinstall
```
> 必须用 altinstall，不要用 install，否则会破坏系统自带的 Python。 系统自带 Python `/usr/bin/python3`不建议卸载。

4. 查看版本
```shell
python3.10 --version
```

5. 想直接用 python、pip 命令
设置别名（永久生效）
```shell
echo "alias python='python3.10'" >> ~/.bashrc
echo "alias pip='pip3.10'" >> ~/.bashrc
source ~/.bashrc
```

