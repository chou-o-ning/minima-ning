---
layout: post
title: "golang 实现的简易 https server 出现 'too many open files' 错误的解决方法"
categories: misc
---

下面是一个很简易的 https server 的 golang 实现，网上常见的一段代码，但其实有一个致命问题。

```
package main

import (
    "io"
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
        io.WriteString(w, "hello, world!\n")
    })
    if e := http.ListenAndServeTLS(":9000", "server.crt", "server.key", nil); e != nil {
        log.Fatal("ListenAndServe: ", e)
    }
}
```

写了一个客户端进行测试，每秒钟访问一次

```
package main

import (
    "crypto/tls"
    "io"
    "log"
    "net/http"
    "os"
    "time"

    "github.com/robfig/cron/v3"
)

func httpGet() {
    c := &http.Client{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
        }}

    if resp, e := c.Get("https://localhost:9000"); e != nil {
        log.Fatal("http.Client.Get: ", e)
    } else {
        defer resp.Body.Close()
        io.Copy(os.Stdout, resp.Body)
    }
}

func main() {
    cn := cron.New(cron.WithSeconds()) //accurate to the second

    // timer
    spec := "*/1 * * * * ?" //Cron Expressions
    cn.AddFunc(spec, httpGet)
    cn.Start()
    select {}
}
```

客户端访问一段时间后，服务端提示错误“http: Accept error: accept tcp [::]:9000: accept4: too many open files”。

用 lsof -p xxxx | grep :9000 | wc -l 命令查看连接个数（xxxx 为 https server 的 pid），发现连接一直在增长，服务端没有把连接释放。

将代码修改如下，设置 ReadTimeout（一分钟），再用 lsof -p xxxx | grep :9000 | wc -l 命令查看连接个数。连接数在一分钟后就不再增长，问题解决。

```
package main

import (
    "io"
    "log"
    "net/http"
    "time"
)

func main() {

    server := &http.Server{
        Addr:         ":9000",
        ReadTimeout:  1 * time.Minute, // 1 min to allow for delays when 'curl' on OSx prompts for username/password
        WriteTimeout: 10 * time.Second,
    }

    http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
        io.WriteString(w, "hello, world!\n")
    })
    if e := server.ListenAndServeTLS("cert.pem", "unencrypted.key"); e != nil {
        log.Fatal("ListenAndServe: ", e)
    }
}
```

因为这个博客使用Jekyll，暂时无法支持评论，因此如果有技术问题，可以发送到我的邮箱一起探讨，邮箱地址为chou dot o dot ning at gmail dot com  

