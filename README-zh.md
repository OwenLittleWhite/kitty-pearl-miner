# Kitty Pearl Miner

一个高性能的 Pearl GPU 矿机。填上池子地址就能挖。

[English](README.md) | 中文

---

## ⚠️ 支持的显卡(先看这里)

本版本**只支持 NVIDIA RTX 30 系**:

| 显卡 | 支持 |
| --- | --- |
| RTX 3060 / 3070 / 3080 / 3090(以及 A10 / A40) | ✅ 支持 |
| RTX 40 系 | ❌ 暂不支持 |
| RTX 50 系 | ❌ 暂不支持 |
| RTX 20 系 / 更老 | ❌ 不支持 |

在不支持的显卡上会报 `no kernel image is available for execution on the device`。
更广的显卡支持在路上。

## 运行要求

- Linux x86-64
- NVIDIA 驱动(支持 CUDA 12 的较新版本)
- CUDA 12 运行时(`libcudart.so.12`)—— 装 CUDA 12 运行时,或把
  `libcudart.so.12` 放二进制旁边、用 `LD_LIBRARY_PATH=.` 运行

用 `nvidia-smi` 看你的显卡。

## 下载

```bash
chmod +x dist/linux-x86_64/pearl-miner
sha256sum -c SHA256SUMS        # 可选:校验
./dist/linux-x86_64/pearl-miner --help
```

## 快速上手

**测算力(不用接池)** —— 默认跑 ~30s,`--secs 60` 跑一分钟:

```bash
./pearl-miner bench
```
```
[bench] GPU 0 (NVIDIA GeForce RTX 3080)  M=16384 N=16384 K=16384 R=64  (~30s)
  57.4 TH/s   (5s)
  57.6 TH/s   (10s)
  ...
sustained 57.5 TH/s   (76.5 ms/iter)
```

**看你的显卡:**

```bash
./pearl-miner list-devices
```
```
  GPU 0: NVIDIA GeForce RTX 3080
  GPU 1: NVIDIA GeForce RTX 3080
```

**挖矿**(默认吃**所有卡**,一个进程驱动每张卡):

```bash
./pearl-miner mine \
  --pool   池子地址:端口 \
  --wallet prl1p你的钱包地址 \
  --worker rig01
# 指定某几张卡:加 --devices 0,1
```
```
[hashrate] 115.0 TH/s  (2 GPUs)
[hashrate] 115.3 TH/s  (2 GPUs)
...
```

每隔几秒打一行汇总 `[hashrate]` 实时算力。

## 参数

**`mine`**

| 参数 | 默认 | 含义 |
| --- | --- | --- |
| `--pool` | *(必填)* | 池子地址 `host:port`。 |
| `--wallet` | *(必填)* | 收益钱包。 |
| `--worker` | `rig` | 矿机 / rig 名。 |
| `--devices` | 所有卡 | 用哪几张卡,如 `0,1,2`。 |
| `--report-secs` | `5` | 滚动算力间隔(秒),`0` 关闭。 |
| `--reconnect-secs` | `5` | 断线后等多少秒重连。 |

**`bench`**

| 参数 | 默认 | 含义 |
| --- | --- | --- |
| `--gpu` | `0` | 测哪张卡。 |
| `--m` `--n` `--k` | `16384` | 工作量大小 —— 越大吞吐越高、显存占用越多。 |
| `--secs` | `30` | 跑多久(如 `--secs 60`)。 |
| `--report-secs` | `5` | 滚动算力间隔。 |

## 性能

**RTX 3080**:默认参数下 **~57–58 TH/s**。

```
./pearl-miner bench                              →  ~57.6 TH/s
./pearl-miner bench --m 8192 --n 8192 --k 8192   →  ~51 TH/s
```

## 常见问题

| 现象 | 解决 |
| --- | --- |
| `libcudart.so.12: cannot open shared object file` | 装 CUDA 12 运行时,或把 `libcudart.so.12` 放二进制旁边并用 `LD_LIBRARY_PATH=.`。 |
| `no kernel image is available for execution on the device` | 你的显卡本版本不支持(仅 RTX 30 系)。 |
| 显存不足 / `ctx create` 失败 | 需要 ~2 GB 空闲显存,调小 `--m/--n/--k`(如 `8192`)。 |
| `pool rejected handshake` | 你的钱包不被池子接受 —— 跟池子运营方核对地址。 |

## 说明

- 一个进程默认驱动你所有的卡;用 `--devices 0,1` 指定某几张,或每张卡起一个
  进程以便用不同的 `--worker` 名。
- `mine` 会一直跑,断线/网络抖动后**自动重连**,可以扔着跑。
