# 压缩Docker镜像的办法

## 问题背景
服务器上存在的docker镜像想要迁移到其他服务器去使用，但是镜像包大小过大，传输不方便。

## 原因分析
Docker 在构建镜像时创建了很多层。压缩有助于在逻辑层中组织镜像。
我们可以控制镜像的结构，而不是让镜像具有多个不必要的层。

## 解决办法
1. 可以使用以下命令安装 `docker-squash`。
```bash
pip install docker-squash
```
2. 运行以下命令来减小镜像的大小。
```bash
docker-squash image:old -t image:new
```
3. 然后使用gzip保存打包镜像
```bash
docker save image:new | gzip > xxx.tar.gz
```
