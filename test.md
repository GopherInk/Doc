部署golang项目时难免要通过命令行来设置一些参数，那么在golang中如何操作命令行参数呢？可以使用os库和flag库。

## golang os库获取命令行参数
os可以通过变量Args来获取命令参数，os.Args返回一个字符串数组，其中第一个参数就是执行文件本身。
```
package main

import (
    "fmt"
    "os"
)

func main() {

    fmt.Println(os.Args)
}
```
$ ./cmd -user="root"编译执行后执行
```
[./cmd -user=root]
```
这种方式操作起来要自己封装，比较费时费劲。golang提供了flag库，可以很方便的操作命名行参数，下面介绍下flag的用法。

## golang flag获取命令行参数

使用flag来操作命令行参数，支持的格式如下：像flag.Int、flag.Bool、flag.String这样的函数格式都是一样的，第一个参数表示参数名称，第二个参数表示默认值，第三个参数表示使用说明和描述。flag.StringVar这样的函数第一个参数换成了变量地址，后面的参数和flag.String是一样的。
```
-id=1
--id=1
-id 1
--id 1
```

```
$ go run flag.go -id=2 -name="golang"
ok: false
id: 2
port: :8080
name: golang
```
使用-h参数可以查看使用帮助：
```
$ go run flag.go -h
-id=0: id
-name="123": name
-ok=false: is ok
-port=":8080": http listen port
```

