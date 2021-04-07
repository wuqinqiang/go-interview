---
title: sync.once
weight: 1
---

# channel

## channel无缓冲和有缓冲的区别？

对于无缓冲的 channel，发送方将阻塞该信道，直到接收方从该信道接收到数据为止，而接收方也将阻塞该信道，直到发送方将数据发送到该信道中为止。

对于有缓存的 channel，发送方在没有空插槽（缓冲区使用完）的情况下阻塞，而接收方在信道为空的情况下阻塞。

## 哪些情况下 channel 会出现 panic ？
- 操作未初始化的 channel
- 向已经关闭的 channel 发送数据
- 关闭已关闭的 channel