# 快速开始

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