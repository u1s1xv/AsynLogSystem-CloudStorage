
# 基于异步日志系统的云存储

一个 C++ 实战项目：异步日志系统 + 云存储服务（类网盘）。

这个项目主要有两个部分：

日志部分：
1. 支持异步写日志，防止写日志阻塞外部业务逻辑
2. 支持备份重要日志，防止crush后无法debug
3. 支持多线程程序并发写日志
4. 支持输出日志到控制台、文件以及按照文件大小滚动文件中，文件大小可配置

存储部分：
1. 是一个类似于网盘的项目
2. 支持浅度存储和深度存储
3. web端上传下载

其实这个项目可以拆成两个项目，一个是异步日志系统，一个是云存储。

为什么要结合一些？

主要是亮点：

* 带大家感受一下，异步日志系统 如何嫁接在另一个系统上
* 项目内容更丰满，是一个完整的全栈实战项目（C++ 后端 + Web 前端）

为什么要做这个项目呢。 首先来聊一聊 日志系统的重要性。

日志系统在软件开发中作用主要在代码编写和调试以及项目启动后的系统运行状况记录。

能够详细记录程序的执行流程、变量的值以及函数的调用情况等，所需要的任何信息都可以通过日志来获取。

由于日志系统在项目的整个生命周期中都有着不可替代的作用，**因此可以说任何项目都可以并且应该集成日志系统以便debug，性能分析等操作**。

也就是说**设计好日志系统 可以嫁接到 所有C++项目里**。

写好异步日志系统，可以嫁接到所有的项目里，为项目添加亮点，毕竟所有的项目都需要打日志！

**如果感觉你自己的项目没有亮点可说，那么就可以在项目里添加个异步日志系统，增加亮点**！

---

# 项目运行指南

## 一、环境依赖

| 依赖 | 版本要求 | 说明 |
|---|---|---|
| g++ | 支持 C++17（g++ 7+） | 编译器 |
| libevent | 2.1.x | HTTP 服务端/客户端网络库（evhttp） |
| jsoncpp | 任意 | JSON 解析/序列化 |
| bundle | 仓库已附带 | 压缩库（`src/server/bundle/libbundle.so`，x86_64 Linux 预编译） |
| pthread / libstdc++fs | g++ 自带 | 线程库与 filesystem |

**Ubuntu/Debian 安装依赖：**

```bash
sudo apt install -y g++ libjsoncpp-dev libevent-dev
```

**如果 apt 没有 libevent-dev 或版本过旧，可源码编译安装（默认安装到 `/usr/local`）：**

```bash
wget https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz
tar xzf libevent-2.1.12-stable.tar.gz
cd libevent-2.1.12-stable
./configure && make && sudo make install
```

> 源码安装到 `/usr/local` 后，编译时需额外加 `-I/usr/local/include -L/usr/local/lib`，运行时需将 `/usr/local/lib` 加入 `LD_LIBRARY_PATH`。

## 二、编译

### 1. 服务端（云存储 HTTP 服务）

```bash
cd src/server
g++ -o test Test.cpp base64.cpp -std=c++17 -lpthread -lstdc++fs \
    -ljsoncpp -L./bundle -lbundle -levent
# libevent 装在 /usr/local 时，追加：-I/usr/local/include -L/usr/local/lib
```

### 2. 客户端（自动上传工具）

```bash
cd src/client
g++ -o client Test.cpp -std=c++17 -lpthread -levent
# libevent 装在 /usr/local 时，追加：-I/usr/local/include -L/usr/local/lib
```

### 3. 日志系统示例

```bash
cd log_system/examples
g++ -o log_test test.cpp -std=c++17 -lpthread -ljsoncpp
```

### 4. 远程日志备份服务器（可选，用于接收 ERROR/FATAL 日志备份）

```bash
cd log_system/logs_code/backlog
g++ -o backup_server ServerBackupLog.cpp -std=c++17 -lpthread
```

## 三、配置说明

### 1. 服务端配置 `src/server/Storage.conf`

| 字段 | 含义 |
|---|---|
| `server_port` | HTTP 服务端口（默认 8081） |
| `server_ip` | 服务器公网 IP，用于渲染网页中的上传地址（BACKEND_URL） |
| `download_prefix` | 下载 URL 路径前缀（`/download/`） |
| `deep_storage_dir` | 深度存储目录（文件压缩后存放） |
| `low_storage_dir` | 浅度存储目录（文件原样存放） |
| `bundle_format` | 深度存储压缩算法编号（4 = MINIZ），枚举见 `src/server/bundle.h` |
| `storage_info` | 存储元数据持久化文件 |

### 2. 日志系统配置 `log_system/logs_code/config.conf`

| 字段 | 含义 |
|---|---|
| `buffer_size` | 日志缓冲区初始容量（字节） |
| `threshold` | 缓冲区达到该大小后停止倍增扩容 |
| `linear_growth` | 达到阈值后的线性扩容步长（字节） |
| `flush_log` | 落盘策略：`0` 仅写入 C 库缓冲区；`1` 调用 `fflush`；`2` `fflush` + `fsync` 强制刷盘 |
| `backup_addr` / `backup_port` | 远程备份服务器地址与端口（ERROR/FATAL 日志通过 TCP 发送至此） |
| `thread_count` | 日志备份线程池的线程数 |

> ⚠️ 日志配置文件的路径是写死的相对路径 `../../log_system/logs_code/config.conf`，**服务端必须在 `src/server/` 目录下运行**。

## 四、运行

### 1. 启动服务端（核心）

```bash
cd src/server
LD_LIBRARY_PATH=./bundle:/usr/local/lib ./test
```

- 成功后在 8081 端口监听（`0.0.0.0:8081`）
- 验证：`curl http://127.0.0.1:8081/` 应返回 200 且页面包含"星河云盘"
- 浏览器访问：`http://<服务器IP>:8081`（云服务器需在安全组/防火墙放行该端口）
- 服务日志滚动写入 `src/server/logfile/` 目录

### 2. 启动远程日志备份服务器（可选）

```bash
./backup_server 8080   # 端口需与 config.conf 中 backup_port 一致
```

### 3. 启动客户端自动上传

```bash
cd src/client
mkdir -p low_storage deep_storage
./client
```

客户端会循环扫描 `low_storage/`、`deep_storage/` 目录，将新增/修改的文件自动上传到服务器（浅度目录走普通存储，深度目录走压缩存储）。

> ⚠️ 客户端默认连接 `127.0.0.1:8081`（硬编码在 `src/client/Storage.hpp` 顶部 `server_port_g` / `server_ip_g`），上传到远程服务器前请修改这两个常量。

### 4. 运行日志系统示例（体验日志库）

```bash
cd log_system/examples
./log_test
```

运行后日志写入 `log_system/examples/logfile/` 目录（`FileFlush.log` 普通文件 + `RollFile_log*` 滚动文件）。

> 说明：示例中的 ERROR/FATAL 日志会尝试连接 `config.conf` 中配置的备份服务器（`backup_addr`/`backup_port`），未启动备份服务器时控制台会输出"正在尝试重连/connect error"提示，**属正常现象**，不影响本地日志落盘。启动备份服务器（见上方第 2 步）后即可消除。

### 5. 终止服务

```bash
# ① 查看运行中的服务进程与端口占用
ss -tlnp | grep 8081        # 查看 8081 端口被哪个进程占用
ps aux | grep -E "\./test|backup_server|\./client"   # 查看服务端/备份服务器/客户端进程

# ② 按进程名终止
pkill -f "^\./test"         # 终止服务端
pkill -f "backup_server"    # 终止备份服务器
pkill -f "^\./client"       # 终止客户端

# ③ 或按 PID 精确终止（PID 来自第 ① 步）
kill <PID>

# ④ 确认端口已释放
ss -tlnp | grep 8081        # 无输出即已终止
```

> ⚠️ `pkill -f` 按完整命令行匹配，建议用 `^` 锚定开头（如 `"^\./test"`），否则可能误杀命令行中恰好包含该字符串的其他进程。

### 6. 后台守护运行（SSH 断开后仍运行）

```bash
cd src/server
setsid nohup env LD_LIBRARY_PATH=./bundle:/usr/local/lib ./test > /tmp/storage_server.log 2>&1 &
```

- `setsid nohup ... &`：进程脱离终端，关闭 SSH 后服务仍运行
- 启动输出重定向到 `/tmp/storage_server.log`（服务自身的运行日志在 `src/server/logfile/`）
- **服务器重启后需重新执行**；如需开机自启，可配置 systemd 服务

## 五、常见问题

| 问题 | 解决方法 |
|---|---|
| `error while loading shared libraries: libbundle.so / libevent...` | 运行时加 `LD_LIBRARY_PATH=./bundle:/usr/local/lib` |
| 打开页面 404 / 找不到配置 | 确认在 `src/server/` 目录下运行服务端（相对路径依赖） |
| 外网访问不了页面 | 云服务器安全组/防火墙放行 8081 端口，确认 `Storage.conf` 中 `server_ip` 为公网 IP |
| 客户端上传文件名乱码 | 已知问题：客户端发送文件名未做 base64 编码而服务端按 base64 解码，见 `TODO.md` 待办 #1 |
| ERROR/FATAL 日志反复提示连接失败 | 未启动备份服务器，或 `config.conf` 中 `backup_addr`/`backup_port` 配置不符；不影响正常日志落盘 |



