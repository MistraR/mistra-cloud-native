# Base  

当前代码名称 Base  

##1.快速创建代码仓库请使用下方命令
```
sudo docker run --rm -v $(pwd): $(pwd) -w  $(pwd) -e ICODE=xxxxxx cap1573/cap-tool new git.imooc.com/cap1573/base

```
 

##2.根据 proto 自动生成 go 基础代码
```
make proto
```

##3.根据代码编译现有 Go 代码  
```
make build
```
代码执行后会产生 base 二进制文件

##4.编译执行二进制文件
```
make docker
```
编译成功后会自动生成 base:latest 镜像
可使用 docker images | grep base 查看是否生成

##5.本课程使用 go-micro v3 版本作为微服务开发框架
框架地址：https://github.com/asim/go-micro


