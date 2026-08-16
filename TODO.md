# 待办清单（基于代码分析 + 试运行发现）

> 记录于 2026-08-16 试运行验证后。按优先级排列，供后续优化迭代参考。

## 🔴 功能性 Bug

1. **客户端/服务端文件名编码协议不一致**（试运行实测发现）
   - 服务端 `src/server/Service.hpp` Upload 中：`filename = base64_decode(filename);`
   - 客户端 `src/client/Storage.hpp` Upload 中：`evhttp_add_header(..., "FileName", fu.FileName().c_str())` 发送**明文**文件名
   - 结果：客户端上传的文件在服务端落盘为乱码文件名，且 `storage.data` 被写入非 UTF-8 字节、不再是合法 JSON
   - 修复：客户端发送前用 `base64_encode` 编码文件名（参考 `src/server/base64.h` 或 cpp-base64）

2. **`JsonUtil::UnSerialize` 永远返回 false**
   - `log_system/logs_code/Util.hpp`（约 144 行）和 `src/server/Util.hpp`（约 274 行）：解析成功路径也 `return false`
   - 目前调用方忽略返回值"碰巧可用"，但属于隐患，应改为成功时 `return true`

3. **`StorageInfo::NewStorageInfo` 把 atime/mtime 写反**
   - `src/server/DataManager.hpp`：`mtime_ = f.LastAccessTime(); atime_ = f.LastModifyTime();`
   - 影响 ETag 稳定性（atime 随访问变化），间接破坏断点续传的 If-Range 校验

4. **`Download` 处理器多个错误**（`src/server/Service.hpp`）
   - 未检查 `GetOneByURL` 返回值，URL 不存在时会用未初始化的 `StorageInfo`
   - 文件缺失分支 `evhttp_send_reply` 后没有 `return`，会重复发送响应
   - 206"断点续传"实际仍发送整个文件：未解析 `Range` 头、未设置 `Content-Range`
   - 404 响应体拼接了 `"not exists"` 到路径字符串

5. **`RollFileFlush::CreateFilename` 时间字段错误**（`log_system/logs_code/LogFlush.hpp`）
   - `tm_hour + 1`、`tm_min + 1`、`tm_sec + 1` 均多加了 1（只有 `tm_mon` 需要 +1）
   - 实测 23:24:58 生成的文件名为 `...16242559-1.log`（时/分/秒全部错位）

6. **ERROR/FATAL 远程备份同步阻塞调用线程**
   - `AsyncLogger::serialize` 中 `tp->enqueue(start_backup, data).get()` 会等待网络备份完成（每次最多 5 次重连）
   - 违背"异步日志不阻塞业务"的设计初衷；建议改为纯异步（不 `.get()`）或可配置开关

## 🟡 设计/健壮性

7. **`Manager.hpp` 缺少 `#pragma once`**（第一行直接是 include，靠包含顺序避免重复定义）

8. **日志库与宿主应用耦合**：`AsyncLogger.hpp` 引用 `extern ThreadPool *tp`，备份功能无法独立开关

9. **硬编码相对路径**：日志配置 `../../log_system/logs_code/config.conf`、服务端 `Storage.conf`/`index.html` 均依赖运行目录（必须在 `src/server/` 下启动）

10. **`evbuffer_copyout` 向 `content.c_str()` 写入**（`Service.hpp`）：对 const 指针强转写入是未定义行为，应改用 `&content[0]`

11. **`Message::format()` 输出缺 `]`**：`[时间][线程id[LEVEL]...`，线程 id 后少一个 `]`

12. **客户端硬编码服务器地址**：`src/client/Storage.hpp` 中 `server_port_g = 8081`、`server_ip_g = "127.0.0.1"`，与 `Storage.conf` 的公网 IP 不一致且不可配置

13. **服务端/客户端文件系统 API 不一致**：服务端用 `std::experimental::filesystem`，客户端用标准 `std::filesystem`

## 🟢 其他

14. 仓库文件模式 644→755 变更（不影响功能，可统一 chmod 还原）
15. `Storage.conf` 中 `server_ip` 为公网 IP `43.172.87.114`，`config.conf` 中 `backup_addr` 同；部署到新环境需确认
16. 编译服务端需链接 libevent（本机位于 `/usr/local`）：`g++ ... -I/usr/local/include -L/usr/local/lib -levent`

## 🚨 性能/架构问题（2026-08-17 线上事件补充）

17. **Upload 同步阻塞事件循环（严重）**：libevent 单线程事件循环中同步执行 LZMA 压缩，大文件压缩期间整个服务无响应。
    - 实测：上传 188MB mp4（深度存储）压缩耗时约 80 秒，期间页面/下载/上传全部超时
    - 建议：压缩改到独立线程/线程池执行，或改为"先落盘、后异步压缩"两阶段处理

18. **无请求体大小限制**：198MB 请求体全量载入内存（`evbuffer_copyout` → `std::string`），大文件上传内存峰值高
    - 建议：限制请求体大小；流式写入磁盘而非全量载入

19. **已压缩格式（视频/图片/PDF）走深度存储无收益**：LZMA 压不动 H.264 视频（188MB→188MB），白费 CPU 与时间
    - 建议：前端按扩展名提示/自动选择存储类型；或后端按文件类型自动降级为普通存储

20. **下载无解压缓存**：深度存储文件每次下载都重新 LZMA 解压（约 1.2s/10MB），重复下载开销大
    - 建议：按（大小+修改时间）缓存解压结果，或下载时直接传输压缩文件（Content-Encoding）
