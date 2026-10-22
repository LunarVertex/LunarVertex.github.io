---
title: 你好,Jerod
date: 2026-08-17 10:00:00
tags:
  - Architecture
  - Go
---
欢迎来到 Jerod 的博客。

博客基于 Hexo 搭建，使用定制优化的 Claudia 主题，并通过 GitHub Actions 自动构建与部署。

## 探索核心方向

这里将记录我对分布式系统、调度器实现与高性能服务架构的实践与思考：

- **Cloud Native**: Kubernetes Controller、Device Plugin 与调度扩展
- **AI Infrastructure**: vLLM、KV-Cache 管理与模型并行推理优化
- **Systems Engineering**: Go 运行时、高并发网关与缓存系统

> Build systems. Study deeply. Keep moving.

## 代码示例

例如一个轻量级的协程调度与优雅停机逻辑：

```go
package main

import (
	"context"
	"fmt"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer stop()

	fmt.Println("🚀 Cloud service started with Go runtime...")

	select {
	case <-ctx.Done():
		fmt.Println("🛑 Graceful shutdown initiated...")
		time.Sleep(500 * time.Millisecond)
	}
}
```

可以使用 `signal.NotifyContext` 优雅监听系统信号，实现平滑停机。
