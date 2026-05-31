# Kitty Pearl Miner

A high-performance GPU miner for Pearl. Point it at a pool and mine.

English | [中文](README-zh.md)

---

## ⚠️ Supported GPUs (read this first)

This build runs on **NVIDIA RTX 30-series only**:

| GPU | Supported |
| --- | --- |
| RTX 3060 / 3070 / 3080 / 3090 (and A10 / A40) | ✅ yes |
| RTX 40-series | ❌ not yet |
| RTX 50-series | ❌ not yet |
| RTX 20-series / older | ❌ no |

On an unsupported card you'll see `no kernel image is available for execution on
the device`. Wider GPU support is coming.

## Requirements

- Linux x86-64
- NVIDIA driver (recent enough for CUDA 12)
- CUDA 12 runtime (`libcudart.so.12`) — install the CUDA 12 runtime, or drop
  `libcudart.so.12` next to the binary and run with `LD_LIBRARY_PATH=.`

Check your card with `nvidia-smi`.

## Download

```bash
chmod +x dist/linux-x86_64/pearl-miner
sha256sum -c SHA256SUMS        # optional: verify
./dist/linux-x86_64/pearl-miner --help
```

## Quick start

**Benchmark your card (no pool needed)** — runs ~30s; `--secs 60` for a minute:

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

**List your GPUs:**

```bash
./pearl-miner list-devices
```
```
  GPU 0: NVIDIA GeForce RTX 3080
  GPU 1: NVIDIA GeForce RTX 3080
```

**Mine** (uses **all GPUs** by default; one process drives every card):

```bash
./pearl-miner mine \
  --pool   POOL_HOST:PORT \
  --wallet prl1pyourwalletaddress \
  --worker rig01
# specific cards: add --devices 0,1
```
```
[hashrate] 115.0 TH/s  (2 GPUs)
[hashrate] 115.3 TH/s  (2 GPUs)
...
```

An aggregate `[hashrate]` line prints every few seconds.

## Options

**`mine`**

| Flag | Default | Meaning |
| --- | --- | --- |
| `--pool` | *(required)* | Pool address `host:port`. |
| `--wallet` | *(required)* | Your payout wallet. |
| `--worker` | `rig` | Worker / rig name. |
| `--devices` | all GPUs | GPUs to mine on, e.g. `0,1,2`. |
| `--report-secs` | `5` | Rolling hashrate interval, seconds (`0` = off). |
| `--reconnect-secs` | `5` | Wait this long before reconnecting after a drop. |

**`bench`**

| Flag | Default | Meaning |
| --- | --- | --- |
| `--gpu` | `0` | GPU to benchmark. |
| `--m` `--n` `--k` | `16384` | Work size — bigger = more throughput and more VRAM. |
| `--secs` | `30` | How long to run (e.g. `--secs 60`). |
| `--report-secs` | `5` | Rolling hashrate interval. |

## Performance

On an **RTX 3080**: **~57–58 TH/s** at the default settings.

```
./pearl-miner bench                              →  ~57.6 TH/s
./pearl-miner bench --m 8192 --n 8192 --k 8192   →  ~51 TH/s
```

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `libcudart.so.12: cannot open shared object file` | Install the CUDA 12 runtime, or put `libcudart.so.12` next to the binary and use `LD_LIBRARY_PATH=.`. |
| `no kernel image is available for execution on the device` | Your GPU isn't supported by this build (RTX 30-series only). |
| out of memory / `ctx create` fails | Needs ~2 GB free VRAM. Lower `--m/--n/--k` (e.g. `8192`). |
| `pool rejected handshake` | Your wallet isn't accepted by the pool — check the address with your pool operator. |

## Notes

- One process drives all your GPUs by default; use `--devices 0,1` to pick
  specific cards, or run one process per card for separate `--worker` names.
- `mine` runs continuously and **reconnects automatically** after a pool or
  network drop, so you can leave it running.
