---
title: "函数式选项模式"
date: 2022-10-18T19:12:11+08:00
draft: false
---

配置参数可变， 如何设计让未来增加方便， 好维护

```go
package main

import (
	"fmt"
	"time"
)

type Conn struct {
	Host    string
	Port    int
	Timeout time.Duration
	Retry   int
}

func NewConn(
	host string, // 必填参数
	fs ...func(*Conn), // 可选可变参数
) *Conn {
	conn := &Conn{
		Host: host,
	}
	for _, f := range fs {
		f(conn)
	}
	return conn
}

func withPort(port int) func(*Conn) {
	return func(c *Conn) {
		c.Port = port
	}
}

func withTimeout(duration time.Duration) func(*Conn) {
	return func(c *Conn) {
		c.Timeout = duration
	}
}

func withRetry(retry int) func(*Conn) {
	return func(c *Conn) {
		c.Retry = retry
	}
}

func main() {
	con := NewConn(
		"localhost",
		withRetry(3),
		withPort(8080),
		withTimeout(2*time.Second),
	)

	fmt.Printf("%#v", con)
}

```

