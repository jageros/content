---
type: "posts"
author: "jager"
title: "go程序优雅退出"
date: "2021-08-03"
tags: [
"go",
"服务端",
"优雅退出",
]
---

> 服务端程序是持续不断运行的，在停服时就需要等待各种服务关闭后再退出程序，
> 本文将介绍go程序优雅退出目前比较推荐的一种实现方式

<!--more-->

## 原理

通过``go1.16+``的NotifyContext方法和errgroup包实现服务的优雅停止

## 代码

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"github.com/gin-gonic/gin"
	"golang.org/x/sync/errgroup"
	"log"
	"net/http"
	"os/signal"
	"syscall"
	"time"
)

type ginHttp struct {
	server *http.Server
}

func NewGinHttpServer(listenIp string, listenPort int, f func(engine *gin.Engine)) *ginHttp {
	engine := gin.Default()
	f(engine)
	s := &http.Server{
		Addr:    fmt.Sprintf("%s:%d", listenIp, listenPort),
		Handler: engine,
	}

	return &ginHttp{
		server: s,
	}
}

func (g *ginHttp) Serve() <-chan error {
	// chan长度大于等于1， 不然errch不存在读取取，将永远阻塞在这里，造成goroutine泄露
	errch := make(chan error, 1)
	go func() {
		err := g.server.ListenAndServe()
		if err != nil {
			errch <- err
			log.Printf("服务停止：%v", err)
		}
		close(errch)
	}()

	return errch
}

func (g *ginHttp) Stop() error {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
	defer cancel()
	return g.server.Shutdown(ctx)
}

func main() {
	// 创建通过监听信号syscall.SIGINT, syscall.SIGEMT来停止的context
	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGEMT)
	defer cancel()

	// 通过errgroup来运行多个服务
	g, ctx := errgroup.WithContext(ctx)

	// 服务一
	g.Go(func() error {
		tr := time.NewTimer(time.Second * 30)
		select {
		// 监听context是否被取消， 取消则终止服务
		case <-ctx.Done():
			/*
				停止服务操作
				stop()
			*/
			log.Println("收到context取消信号，停止服务一")
			return ctx.Err()

		// 模拟服务
		case <-tr.C:
			// 模拟30秒后服务出错返回， errgroup会对errgroup.WithContext返回的context进行取消
			log.Println("服务一出错： xxx")
			return errors.New("xxx")
		}
	})

	// 服务二： 基于gin框架的web服务
	g.Go(func() error {
		ginSvr := NewGinHttpServer("0.0.0.0", 8080, func(engine *gin.Engine) {
			engine.GET("/hello", func(c *gin.Context) {
				c.String(http.StatusOK, "Hello World!")
			})
		})
		select {
		// 监听context是否被取消， 取消则终止服务
		case <-ctx.Done():
			//停止服务操作
			log.Println("收到context取消信号，停止服务二")
			err := ginSvr.Stop()
			if err != nil {
				log.Printf("停止服务二出错： %v", err)
			} else {
				log.Println("成功停止服务二")
			}
			return ctx.Err()

		// http服务
		case err := <-ginSvr.Serve():
			// 如果服务完成并未出错， errgroup不会对返回的context进行取消操作
			if err != nil {
				log.Printf("服务二出错： %v", err)
			}
			return err
		}
	})

	// 等待所有服务完成，或者某个服务报错并终止所有服务
	err := g.Wait()
	log.Printf("程序退出：%v", err)
}
```
