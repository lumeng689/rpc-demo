# 1. 安装 go dep

## windows

访问 https://github.com/golang/dep/releases，下载 dep-windows-amd64.exe，下载完成将文件拷贝到GOPATH下bin文件夹内。
就是go的安装目录下的bin文件内，改名称为 dep。
新打开命令行， 执行 

```
dep version
```

## mac

```
brew install dep
brew upgrade dep
```

## linux

```bash
sudo apt-get install go-dep
# other platform
curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
```

# 2. 初始化dep

进入项目目录，执行

```bash
dep init
```

## 常用命令

```bash
# 查看状态 
dep status

# 同步
dep ensure

# 建议使用 v=verbose
dep ensure -v

#更新
dep ensure -update

#删除没有用到的 package
dep prune -v

#添加依赖
dep ensure -add github.com/gin-gonic/gin
```

# 3 protoc-gen-go 用法

安装
```bash
# 下载protoc文件 https://github.com/protocolbuffers/protobuf/releases

# 下载protoc-gen-go
go get -u github.com/golang/protobuf/protoc-gen-go
```



```bash
#进入pb目录
cd pb

# 执行命令进行生成
protoc --go_out=. *.proto
# 生成grpc调用代码，得写上 plguins， ： 后面 是输出路径
protoc  helloworld.proto --go_out=plugins=grpc:./
```

# 4 执行demo

运行grpc 服务
```bash
go run grpc/server.go
```

运行 gin 服务
```bash
go run gin/main.go
```

浏览器中访问
http://localhost:8052/rest/n/zhangsan

# 5 protoc-gen-grpc-web 安装

## windows
访问 https://github.com/grpc/grpc-web/releases 下载 protoc-gen-grpc-web-1.0.4-windows-x86_64.exe(最新版本即可)，将其放到
windows的执行路径里，更改名称为protoc-gen-grpc-web.exe

## mac

从上面的地址下载 protoc-gen-grpc-web-1.0.4-darwin-x86_64，然后执行：
```bash
sudo mv ~/Downloads/protoc-gen-grpc-web-1.0.4-darwin-x86_64 /usr/local/bin/protoc-gen-grpc-web
chmod +x /usr/local/bin/protoc-gen-grpc-web
```

然后执行命令进行生成即可
```
protoc -I=$DIR echo.proto \
--js_out=import_style=commonjs:$OUT_DIR \
--grpc-web_out=import_style=commonjs,mode=grpcwebtext:$OUT_DIR
```

生成Typescript例子
```bash
protoc -I=./ echo.proto \
--js_out=import_style=commonjs:./ \
--grpc-web_out=import_style=typescript,mode=grpcwebtext:./
```

$DIR 替换为 echo.proto所在目录
$OUT_DIR 代表输出目录， 注意， 第二行和第三行都有

输出为typescript的话，更改第三行代码的import_style 为 import_style=commonjs+dts或者 import_style=typescript

# 6 Docker 命令

```bash
# 查看宿主机IP Docker Toolbox 上访问，不能使用localhost，要用这个地址
docker-machine ip default

# 查看镜像
docker images

# 删除镜像, 删除之前要先把 容器都删除  xxxxx 代表容器ID
docker rmi xxxxx

# 查看当前运行的容器
docker ps

# 删除容器 正在运行的容器无法删除，需要使用docker stop 或者docker kill
docker rm xxxxx

# 删除所有容器
docker rm `docker ps -a -q`

# 进入容器内部
docker exec -it xxxxx /bin/bash

# 启动已关闭的容器
docker start xxxxx

# 查看容器内日志
docker logs xxxxx

# 查看容器内进程
docker top xxxxx

# 关闭容器
docker stop xxxxx
docker kill xxxxx

# 获取详细的容器信息
docker inspect xxxxx

```

# 7 执行grpc/grpc-web项目的demo时问题解决

## 下载
```bash
https://github.com/grpc/grpc-web
```

## 启动
切换到grpc-web目录下

首先 修改 grpc-web/net/grpc/gateway/examples/echo/envoy.yaml文件
删除34行的enabled: true，替换为以下内容：
```yaml
filter_enabled: 
    default_value:
        numerator: 100
        denominator: 0
    runtime_key: "1"
```

![如下](/static/pic1.png)

```bash
# 执行目录要在grpc-web根目录

# 构建基础服务
docker build -t grpcweb/common -f net/grpc/gateway/docker/common/Dockerfile .

#  启动后台服务
docker build -t grpcweb/node-server -f net/grpc/gateway/docker/node_server/Dockerfile .
docker run -d -p 9090:9090 --name node-server grpcweb/node-server

# 启动Envoy代理服务
docker build -t grpcweb/envoy -f net/grpc/gateway/docker/envoy/Dockerfile .
docker run -d -p 8080:8080 --link node-server:node-server grpcweb/envoy

# 启动前端服务
docker build -t grpcweb/commonjs-client -f net/grpc/gateway/docker/commonjs_client/Dockerfile .
docker run -d -p 8081:8081 grpcweb/commonjs-client

```

访问本地浏览器 [http://localhost:8081/echotest.html](http://localhost:8081/echotest.html)
windows 上如果使用 Docker Toolbox，则将localhost改成 192.168.99.100 这个地址是执行  docker-machine ip default取得的。