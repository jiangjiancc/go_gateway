<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*


# Go 微服务网关代码使用说明


## 代码帮助

### 运行后端代码

- 首先git clone 本项目

`git clone git@github.com:e421083458/gateway_demo.git`

- 确保本地环境安装了Go 1.12+版本

```
go version
go version go1.12.15 darwin/amd64
```

- 下载类库依赖

```
export GO111MODULE=on && export GOPROXY=https://goproxy.cn
cd gateway_demo
go mod tidy
```

- 在相应功能文件夹下，执行 `go run main.go` 即可。


### 运行后端项目

- 首先git clone 本项目

`git clone git@github.com:e421083458/go_gateway.git`


- 确保本地环境安装了Go 1.12+版本

```
go version
go version go1.12.15 darwin/amd64
```

- 下载类库依赖

```
export GO111MODULE=on && export GOPROXY=https://goproxy.cn
cd go_gateway
go mod tidy
```

- 创建 db 并导入数据

```
mysql -h localhost -u root -p -e "CREATE DATABASE go_gateway DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;"
mysql -h localhost -u root -p go_gateway < go_gateway.sql --default-character-set=utf8
```

- 调整 mysql、redis 配置文件

修改 ./conf/dev/mysql.toml 和 ./conf/dev/redis.toml 为自己的环境配置。

- 运行面板、代理服务

运行管理面板配合前端项目 - 达成服务管理功能
```
go run main.go -config=./conf/dev/ -endpoint dashboard
```

运行代理服务
```
go run main.go -config=./conf/dev/ -endpoint server
```

### 运行前端项目

- 首先git clone 本项目

```
git clone git@github.com:e421083458/go_gateway_view.git
```

- 确保本地环境安装了nodejs

```
node -v
v11.9.0
```

- 安装node依赖包

```
cd go_gateway_view
npm install
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install
```

- 运行前端项目

```
npm run dev
```

### 后端goland编辑器参考

- 用 goland 打开项目目录

- 设置 goland 支持 go mod
    - Preferences-> Go-> Go Modules（vgo）
    - 勾选 Enable Go Modules（vgo）
    - proxy 设置：https://goproxy.cn

- 在相应文件夹下的main方法中， 点击 `run go build` 即可。


### vscode编辑器设置参考

- 用 vscode 打开前端项目目录

- 安装格式化插件 ESLint、Vetur、vue-beautify

## 代码部署

说明一下：
go_gateway_demo_view 与 go_gateway_demo 是视频时开发演示的项目。

实际放置到 github上是  go_gateway_view 与 go_gateway

### 实体机部署
#### 1、每个项目独立部署
- 前端项目一个端口
- 接口项目一个端口
- 使用nginx将后端接口设置到跟前端同域下访问
```
    server {
        listen       8882;
        server_name  localhost;
        root /Users/niuyufu/VueProjects/go_gateway_view/dist;
        index  index.html index.htm index.php;

        location / {
            try_files $uri $uri/ /index.html?$args;
        }

        location /prod-api/ {
            proxy_pass http://127.0.0.1:8880/;
        }
    }

```
- 代理服务器独立部署
- 后端项目启动脚本，所有后端只需要一个脚本了：vim onekeyupdate.sh

#### 2、前后端合并部署
- 前端打包dist放到后端同一项目中
- 后端设置: vim http_proxy_router/route.go
```
router.Static("/dist", "./dist")
```
- 启动接口项目
- 启动代理服务器，所有项目只需要一个脚本了: vim onekeyupdate.sh

### k8s部署

- 创建docker文件 vim dockerfile_dashboard
- 创建docker镜像：
```
docker build -f dockerfile_dashboard -t dockerfile_dashboard .
```
- 运行测试docker镜像: 
```
docker run -it --rm --name go_gateteway_dashboard go_gateteway_dashboard
```
- 创建交叉编译脚本，解决build太慢问题  vim docker_build.sh
- 编写服务编排文件，vim k8s_dashboard.yaml
- 启动服务
```
kubectl apply -f k8s_dashboard.yaml
kubectl apply -f k8s_server.yaml
```
- 查看所有部署
```
kubectl get all
```
