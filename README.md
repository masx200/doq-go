# doq-go

<div align="center">

[![Release](https://img.shields.io/github/release/masx200/doq-go/all.svg)](https://github.com/masx200/doq-go/releases)
[![Go version](https://img.shields.io/github/go-mod/go-version/masx200/doq-go)](https://github.com/masx200/doq-go/blob/master/go.mod#L3)
[![](https://godoc.org/github.com/masx200/doq-go/doq?status.svg)](https://godoc.org/github.com/masx200/doq-go/doq)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![masx200](https://circleci.com/gh/masx200/doq-go/tree/main.svg?style=svg)](https://circleci.com/gh/masx200/doq-go?branch=main)
[![lint](https://github.com/masx200/doq-go/actions/workflows/lint.yml/badge.svg?branch=main)](https://github.com/masx200/doq-go/actions/workflows/lint.yml)
[![codecov](https://codecov.io/gh/masx200/doq-go/branch/main/graph/badge.svg?token=77659YBXM8)](https://codecov.io/gh/masx200/doq-go)
[![Go Report Card](https://goreportcard.com/badge/github.com/masx200/doq-go)](https://goreportcard.com/report/github.com/masx200/doq-go)

**DNS over QUIC (DoQ) 客户端库**

基于 [quic-go](https://github.com/quic-go/quic-go) 和 [dns](https://github.com/miekg/dns) 库构建的 Go 语言 DNS over QUIC 客户端库，完全符合 [RFC9250](https://datatracker.ietf.org/doc/rfc9250/) 规范。

</div>

## 目录

- [功能特性](#功能特性)
- [安装](#安装)
- [快速开始](#快速开始)
- [示例](#示例)
- [配置选项](#配置选项)
- [更新日志](#更新日志)
- [贡献](#贡献)
- [许可证](#许可证)

## 功能特性

- ✅ 完全符合 RFC9250 规范
- ✅ 高性能 QUIC 传输
- ✅ 支持 TLS 1.3 加密
- ✅ 简洁易用的 API
- ✅ 支持上下文取消
- ✅ 完整的错误处理
- ✅ 支持自定义 DNS 服务器

## 安装

使用 go get 添加依赖：

```bash
go get github.com/masx200/doq-go
```

## 快速开始

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/masx200/doq-go/doq"
    "github.com/miekg/dns"
)

func main() {
    // 创建使用默认设置的客户端，通过 AdGuard DoQ 服务器解析
    client := doq.NewClient("dns.adguard-dns.com:853")

    // 准备查询负载
    q := dns.Msg{}
    q.SetQuestion("www.google.com.", dns.TypeA)

    // 发送 DNS 查询
    r, err := client.Send(context.Background(), &q)
    if err != nil {
        log.Fatal(err)
    }

    // 处理响应
    fmt.Println("响应代码:", dns.RcodeToString[r.Rcode])
    for _, ans := range r.Answer {
        fmt.Println(ans.String())
    }
}
```

## 示例

### 基本查询

```go
// 创建客户端
client := doq.NewClient("dns.adguard-dns.com:853")

// 查询 A 记录
q := dns.Msg{}
q.SetQuestion("example.com.", dns.TypeA)

// 发送查询
r, err := client.Send(context.Background(), &q)
if err != nil {
    panic(err)
}

// 处理响应
fmt.Println(dns.RcodeToString[r.Rcode])
```

### 使用自定义选项

```go
// 使用自定义选项创建客户端
client := doq.NewClient("dns.quad9.net:853",
    doq.WithTimeout(5*time.Second),
    doq.WithTLSConfig(&tls.Config{
        ServerName: "dns.quad9.net",
    }),
)

// 查询 AAAA 记录
q := dns.Msg{}
q.SetQuestion("ipv6.google.com.", dns.TypeAAAA)

// 使用超时上下文
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

r, err := client.Send(ctx, &q)
if err != nil {
    panic(err)
}
```

### 并发查询

```go
client := doq.NewClient("dns.adguard-dns.com:853")

domains := []string{"google.com", "github.com", "stackoverflow.com"}

var wg sync.WaitGroup
for _, domain := range domains {
    wg.Add(1)
    go func(d string) {
        defer wg.Done()
        
        q := dns.Msg{}
        q.SetQuestion(d+".", dns.TypeA)
        
        r, err := client.Send(context.Background(), &q)
        if err != nil {
            fmt.Printf("查询 %s 失败: %v\n", d, err)
            return
        }
        
        fmt.Printf("%s: %s\n", d, dns.RcodeToString[r.Rcode])
    }(domain)
}
wg.Wait()
```

## 配置选项

doq-go 提供了多种配置选项来自定义客户端行为：

```go
client := doq.NewClient("dns.example.com:853",
    doq.WithTimeout(10*time.Second),           // 设置超时时间
    doq.WithTLSConfig(customTLSConfig),        // 自定义 TLS 配置
    // 更多选项...
)
```

## 更新日志

### v0.55.0

- 🚀 **升级**: 升级 `github.com/quic-go/quic-go` 到 v0.55.0
- 🐛 **修复**: 修复了因 quic-go 升级导致的类型错误
- ⚠️ **重大变更**: 适配了 quic-go 的 breaking changes
- 🔧 **改进**: 优化了错误处理和连接管理

### 早期版本

- 初始版本发布
- 实现基本的 DoQ 客户端功能
- 添加 TLS 支持和上下文取消

## 贡献

欢迎贡献代码！请遵循以下步骤：

1. Fork 本仓库
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开一个 Pull Request

## 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 致谢

- [quic-go](https://github.com/quic-go/quic-go) - QUIC 协议的 Go 实现
- [dns](https://github.com/miekg/dns) - DNS 库的 Go 实现
- [RFC9250](https://datatracker.ietf.org/doc/rfc9250/) - DNS over Dedicated QUIC Connections 规范

---

<div align="center">

**如果这个项目对您有帮助，请给一个 ⭐️**

</div>
