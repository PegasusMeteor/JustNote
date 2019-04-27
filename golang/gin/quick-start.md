# 快速开始

## 初始化gin工程

1、下载并安装gin

```shell
go get -u github.com/gin-gonic/gin
```

2、安装一个go工程的依赖管理工具例如 Govendor 或者dep，我们这里使用dep. 安装方法可以参考 [https://github.com/golang/dep](https://github.com/golang/dep)  

3、创建一个go web 工程,初始化包依赖管理

```shell
mkdir -p $GOPATH/src/github.com/PegasusMeteor/goweb-gin && cd "$_"
```

```shell
dep init
```

4、创建一个main.go 然后将下面代码贴进去，然后运行，这样一个quick start demo，就完成。

```go
package main

import (
    "github.com/gin-gonic/gin"
)


func setupRouter() *gin.Engine {
    // 默认命令行的log输出是有颜色的，如果想要关闭可以执行下面的设置
    // gin.DisableConsoleColor()
    r := gin.Default()

    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    return r
}

func main() {
    r := setupRouter()
    //默认端口是8090
    r.Run(":8090")
}

```

```go
go run main.go
```

命令行或者浏览器访问 http://localhost:8090/ping 就可以啦。

## 初始化前端工程 Ant Design Pro

Ant Design Pro 的安装比较简单，可以参考官方repo. [Ant Design Pro](https://github.com/ant-design/ant-design-pro/blob/master/README.zh-CN.md)

```shell
$ git clone https://github.com/PegasusMeteor/ant-design-pro.git --depth=1
$ cd ant-design-pro
$ npm install
$ npm start         # 访问 http://localhost:8000
```

安装的过程可能会花费一点时间，安装完成之后，直接在浏览器界面访问 http://localhost:8000 就可以了。如果网络环境很差劲的话，可以尝试使用cnpm来解决问题。

**备注**，为了避免官方代码更新频率过快，对我们学习过程中的使用事例造成影响，我对官方库进行了fork，以后的修改都将在此库的基础上进行。
