# An ebpf package manager

- this is an design or demo, see [lmp](https://github.com/linuxkerneltravel/lmp) project for details.

## index

- [bindsnoop](eBPF_hub/bindsnoop)
- [bootstrap](eBPF_hub/bootstrap)
- [fentry-link](eBPF_hub/fentry-link)
- [kprobe-link](eBPF_hub/kprobe-link)
- [mdflush](eBPF_hub/mdflush)
- [lsm-connect](eBPF_hub/lsm-connect)
- [minimal](eBPF_hub/minimal)
- [mountsnoop](eBPF_hub/mountsnoop)
- [opensnoop](eBPF_hub/opensnoop)
- [sigsnoop](eBPF_hub/sigsnoop)
- [template](eBPF_hub/template)
- [tcpconnlat](eBPF_hub/tcpconnlat)

> index can be auto generated.

## design

### 一个包管理器：ebpm

类似于 cargo 和 wapm

- wapm 包含 ecli 的部分。这部分可以用 go 写；
- 专注于一个部分：获取 ebpf 数据文件（本质上是一个分布式文件版本管理系统），看看能不能复用 git
- 可以做成分布式、去中心化的，使用 url 进行定位；

用例：

- 角色1：普通用户/user

首先，我们有一个开发人员的用例，他想使用 ebpf 二进制文件或者程序，但不知道如何/在哪里找到它:

试着运行

```bash
./ebpm run opensnoop                                                     # 使用一个名字直接跑起来
./ebpm run https://github.com/ebpf-io/raw/master/examples/opensnoop.bpf  # 使用一个http API
./ebpm run ./opensnoop.bpf                                               # 使用一个本地路径
```

```bash
ECLI_REGISTRY="https://ebpf-registry.example.com"
ecli run "package-name"
->
curl "$ECLI_REGISTRY/{package-name}.json" | ecli run
```

- 角色2：通用 ebpf 数据文件发布者/ebpf developer

我们的第二个角色是一个开发人员，他想要创建一个通用二进制，并在任何机器和操作系统上分发它。这对于命令行工具或者可以直接在Shell中运行的任何东西都很有用:

create project

```console
$./ebpm init opensnoop
$ cd opensnoop
$ tree -a
.
├── bootstrap.bpf.c
├── bootstrap.bpf.h
├── config.json
├── .gitignore
└── README.md
$./ebpm build opensnoop
```

- 需要有约束，gcc 和 linux 版本；

会产生一个配置文件模板：

```toml
[package]
name = "username/my_package"
version = "0.1.0"
description = ""
license = "MIT"

[[module]]
name = "my_app"
source = "path/to/app.ebpf"
```

发布 ebpf 数据文件

```bash
./ebpm publish opensnoop
```

我应该在哪里发布它？Github？Npm？但这只是 ebpf，没有任何语言的关联…那就Github！

git push ...

- 角色3：其他程序的开发者/ebpf 程序使用者/other developers

可以直接下载：

我们可以在任何有绑定的语言中使用 ebpf：

```bash
./ebpm get opensnoop
```

这会创建一个 config 文件；

或者在 config 里面定义之后：

ebpm.toml/json
```c
[[module]]
name = "opensnoop"、
path = ”http://....."
version = 0.1

[[module]]
name = "execsnoop"
path = ”./bpftools/execsnoop.bpf”
version = 0.1
```

运行

```bash
./ebpm install .
```

就能在本地下载并运行；

```go
import "ebpm"

handler := ebmp.open_and_run("execsnoop")
handler.stop()
handler := ebmp.open_and_run("execsnoop")
handler.stop()
```

或者更进一步，它应该可以被内嵌在别的包管理器里面，比如，我想安装一个 go 的 opensnoop 包，我只需要：

```bash
go get ebpm-opensnoop
```

```go
import "ebpm-opensnoop"
```

所有这些用例促使我们重新思考包管理器的当前全景，以及我们如何创建一个只关注 ebpf 的包管理器，它将统一以下原则:

- 它应该使发布、下载和使用 ebpf 模块变得容易；
- 它应该支持在 ebpf 之上定义命令的简单方法；
- 它应该允许不同的ABI：甚至未来的新ABI。
- 它应该可以嵌入到任何语言生态中(Python、PHP、Ruby、JS…)，而不会强迫一个生态进入另一个生态

需要注意循环依赖；
有必要的话，某些库可以有供应商依赖；

- 直接从GitHub，BitBucket，GitLab，托管Git和HTTP中提取依赖项
- 完全可重现的构建和依赖性解析
- 完全分散 - 没有中央服务器或发布过程
- 允许任何构建配置
- 私有和公共依赖，以避免“依赖地狱”
- 每个包有多个库，因此像Lerna这样的工具是不必要的
- 将单个包装从单体仓库中取出
- 完全支持语义版本控制
- 通过直接依赖于Git分支来快速移动，但是以受控方式
- 版本等效性检查以减少依赖性冲突
- TOML配置文件，便于计算机和人员编辑
- 离线工作
- 只需单击一下即可使所有内容保持最新状态
