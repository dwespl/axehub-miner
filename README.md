# AxeHub Miner

[![Latest release](https://img.shields.io/github/v/release/dwespl/axehub-miner?color=ff8a1e&label=release)](../../releases/latest)
[![Downloads](https://img.shields.io/github/downloads/dwespl/axehub-miner/total?color=ff8a1e&label=downloads)](../../releases)
![Platforms](https://img.shields.io/badge/platforms-Windows%20%7C%20Linux%20%7C%20Hive%20OS-54a8ff)
![GPU backends](https://img.shields.io/badge/GPU-CUDA%20%7C%20OpenCL%20%7C%20Vulkan%20%7C%20CPU-3fd07a)
![License](https://img.shields.io/badge/license-proprietary%20(binary--only)-8593a5)

A fast multi-backend **solo miner** for six proof-of-work algorithms from one binary:

- **CapStash (CAP)** — whirlpool (default)
- **BitcoinIII (BC3)** — bc3hash (`--algo bc3hash`; the pre-fork `sha3-256t` still works)
- **Quantus** — qpow (`--algo qpow`)
- **Numen** — poscan (`--algo poscan`; NVIDIA via CUDA, AMD via OpenCL on Linux)
- **RIGOR** — rgrpow01 (`--algo rgrpow01`; NVIDIA via CUDA, AMD via OpenCL or Vulkan)

It mines on NVIDIA (CUDA), AMD (OpenCL or Vulkan), Intel &
other GPUs (Vulkan), or CPU (x86-64), on **Windows, Linux and Hive OS**. Live TUI + HTTP status, bit-exact GPU==CPU self-test,
a built-in tuner, Prometheus metrics, 1% disclosed dev fee.

There is no pool hardcoded anywhere: point `--pool` at any pool serving these
coins. Ours are `pool.axehub.app:3336` (CAP) and `pool.axehub.app:3338` (BC3),
and they are what the examples below use.

> Distributed in **binary form only**. See [Releases](../../releases) for the
> latest build.

## What it does

- **Four backends in one binary:** CUDA, OpenCL, Vulkan and CPU. `--list-algos` prints
  each algorithm with its backends, `--list-devices` the cards the miner sees. whirlpool,
  sha3-256t and qpow run on any backend; bc3hash on NVIDIA (CUDA), other GPUs (OpenCL) and
  the CPU; poscan on NVIDIA (CUDA) and AMD (OpenCL, Linux) only; rgrpow01 on NVIDIA (CUDA)
  and AMD (OpenCL or Vulkan).
- **Live control while mining:** a keyboard panel and a built-in HTTP page — pause per
  device, change the pool, wallet or worker on the fly, save to the config, restart.
  Press `h`, or use **Hide IDs** on the page, to blank the pool, wallet and worker for a
  screenshot.
- **Monitoring:** XMRig-compatible `/1/summary` and `/2/summary` and a Prometheus
  `/metrics` endpoint, ready for existing dashboards.
- **Built-in tuner:** `--tune hashrate|efficiency|balanced|profit` sets the core and
  memory clock on NVIDIA and the power cap on AMD on Linux, saves the point beside the
  config and re-applies it at start. `--tune performance` (NVIDIA) instead applies a safe
  automatic core-clock overclock — each step verified bit-exact — at the same power limit.
- **Correctness check:** `--selftest` proves every GPU kernel bit-exact against a CPU
  reference.
- **Pool-agnostic:** no hardcoded pool; `--fallback` takes one or more backup pools and
  the miner fails back to the first.
- **Rig safety:** a per-device temperature limit with automatic pause and resume, plus an
  optional watchdog and emergency power-off, both off by default.
- **Diagnostics:** `--log-file` with rotation, and `--report` for a bundle with the wallet
  and password hidden.

## Download

Grab the archive for your OS from the [latest release](../../releases/latest):

| OS | Archive | Backends |
|----|---------|----------|
| Windows x64 | `axehub-miner-v2.0.2-windows-x64.zip` | CUDA · OpenCL · Vulkan · CPU |
| Linux x64 — NVIDIA and CPU | `axehub-miner-v2.0.2-linux-x64.tar.gz` | CUDA · OpenCL · Vulkan · CPU |
| Linux x64 — AMD | `axehub-miner-v2.0.2-linux-x64-amd.tar.gz` | CUDA · OpenCL · Vulkan · CPU, with the AMD OpenCL runtime for bc3hash, qpow and poscan |
| Hive OS     | `axehub-miner-2.0.2.tar.gz` | CUDA · OpenCL · Vulkan · CPU |

To mine bc3hash or poscan on an AMD card on Linux, take the **AMD** package. Everyone else
on Linux wants the smaller one. whirlpool and sha3-256t run on AMD there through Vulkan;
qpow uses the bundled OpenCL runtime in the AMD package (Vulkan as a fallback); poscan on
AMD runs only through that OpenCL runtime (no Vulkan form).

## Run with Docker

Three images on GHCR wrap the same release binary — one per GPU vendor. The
**[one-command setup wizard](https://axehub.app/start/)** (also in 中文, Español,
Português and Русский) writes the exact `docker run` line for your card, algorithm
and wallet; the images work directly too:

| Image | For |
|---|---|
| `ghcr.io/dwespl/axehub-miner:nvidia` | NVIDIA GPUs (CUDA) — needs the NVIDIA Container Toolkit |
| `ghcr.io/dwespl/axehub-miner:amd` | AMD GPUs (OpenCL) — exposes `/dev/dri` + `/dev/kfd` |
| `ghcr.io/dwespl/axehub-miner:cpu` | any x86-64 host, CPU only |

Each also has an immutable `:VERSION-<variant>` tag (e.g. `:2.0.2-nvidia`); images are `linux/amd64`.

```sh
# NVIDIA — set wallet and algorithm; the pool defaults to AxeHub's own
docker run -d --name axehub-miner --restart unless-stopped --gpus all \
  -e WALLET=cap1qYOUR_ADDRESS -e ALGO=whirlpool \
  ghcr.io/dwespl/axehub-miner:nvidia

# AMD, BitcoinIII (bc3hash) — the cache volume keeps the one-time OpenCL compile
docker run -d --name axehub-miner --restart unless-stopped \
  --device /dev/dri --device /dev/kfd -v axehub_cache:/root/.cache \
  -e WALLET=bc1qYOUR_ADDRESS -e ALGO=bc3hash \
  ghcr.io/dwespl/axehub-miner:amd

# CPU only
docker run -d --name axehub-miner --restart unless-stopped \
  -e WALLET=cap1qYOUR_ADDRESS -e ALGO=whirlpool \
  ghcr.io/dwespl/axehub-miner:cpu
```

`WALLET` and `ALGO` are all you need; `POOL` defaults per algorithm (`whirlpool`
and `bc3hash` to AxeHub's pool, `sha3-256t`, `qpow`, `poscan` and `rgrpow01` need `-e POOL=host:port`).
Also accepted: `WORKER`, `PASS`, `BACKEND`, `FALLBACK`, `HTTP`, `EXTRA_ARGS`,
`CONFIG`. Watch it with `docker logs -f axehub-miner`; stop with `docker stop axehub-miner`.

The AMD image compiles its OpenCL kernel on the first start (a few minutes — keep
the cache volume). Confirm an NVIDIA host is set up with:

```sh
docker run --rm --gpus all --entrypoint /opt/axehub/axehub-miner \
  ghcr.io/dwespl/axehub-miner:nvidia --selftest   # prints: self-test PASS
```

On Windows the CPU image runs through Docker Desktop/WSL2, and NVIDIA passthrough
works there too, though the native zip stays the proven path for GPU mining on
Windows; for an AMD card on Windows use the zip. macOS and ARM are not supported —
use a Linux x86-64 host or the native packages.

## Quick start

**Windows**
1. Extract the zip.
2. Open `axehub-miner.conf` and set `wallet` to your payout address — CapStash
   (`cap1...`) by default, or BitcoinIII (`bc1...`) with `algo = bc3hash` and the
   BC3 pool (see *Mining BitcoinIII (BC3)* below).
3. Run `axehub-miner.exe`.

**Linux**
```sh
tar -xzf axehub-miner-v2.0.2-linux-x64.tar.gz
cd axehub-miner-v2.0.2-linux-x64
chmod +x axehub-miner libcap_cuda.so
# edit axehub-miner.conf -> set your wallet (CapStash cap1... by default;
# BitcoinIII: bc1... + algo = bc3hash)
./axehub-miner
```

**Linux, an AMD card (bc3hash shown; qpow and poscan start the same way)**
```sh
tar -xzf axehub-miner-v2.0.2-linux-x64-amd.tar.gz
cd axehub-miner-v2.0.2-linux-x64-amd
# edit axehub-miner.conf -> wallet = bc1..., algo = bc3hash,
# pool = pool.axehub.app:3338, password = x
sudo ./run.sh
```
`run.sh` sets up the bundled OpenCL runtime for this process only. The first run needs
root to install the GPU builtins to `/usr/lib/clc`, and the card needs its render node, so
later runs need root or membership of the `render` group. Requires glibc 2.35 or newer
(Ubuntu 22.04, Debian 12 or newer).

A live status panel appears; the pool credits blocks to your address (solo).
Its keys: `p` pause/resume (`0`-`9` one device), `w` wallet, `o` pool, `k` worker,
`t` temperature limit, `s` save those to the config, `d` details, `u` tune,
`e` tune mode, `r` restart, `h` hide IDs, `q` stop. The `--http` status page has the same controls.

**Hive OS**

1. **Wallets → Coins → Add Coin** — add a custom coin (`CapStash` or
   `BitcoinIII`); the algorithm you pick there is only a Hive label, the miner
   takes its algorithm from the **Hash algorithm** field below. Then **Add
   Wallet** for that coin with your `cap1...` / `bc1...` address. The template
   field also accepts a literal address instead of `%WAL%`.
2. Create a Flight Sheet, pick **Custom** as the miner, and set the pool to
   **Configure in miner**.
3. **Installation URL** — the URL of `axehub-miner-2.0.2.tar.gz` from the
   [latest release](../../releases/latest). **Miner name** — `axehub-miner`.
4. Fill in the flight sheet:

   | Field | Value |
   |---|---|
   | Pool URL (required) | `host:port` — the port is required. `pool.axehub.app:3336` for CAP, `pool.axehub.app:3338` for BC3 |
   | Wallet and worker template (required) | `%WAL%.%WORKER_NAME%` |
   | Hash algorithm | `whirlpool` for CAP (the default if left blank) — **required for BC3**: set `bc3hash` (the pre-fork `sha3-256t` is still accepted); `qpow` for qpow pools; `poscan` for poscan pools |
   | Pass | optional — `d=1` asks the pool for fixed difficulty 1 (whirlpool only; for bc3hash use `x`) |
   | Extra config arguments | optional tuning keys, one per line |

5. Apply the flight sheet to your rig.

A second `host:port` in the same Pool URL field, separated by a space, is used as
a fallback; it must serve the same coin (a second line is ignored). Extra config
keys are appended last and override earlier values, so keep the pool, wallet and
algorithm in their own fields. `temp_limit` applies to any card whose temperature the
miner can read.

Hashrate, accepted/rejected shares and uptime report to the Hive OS dashboard.
Per-GPU temperature and fan are read from NVML on NVIDIA and from the kernel's
`hwmon` node on AMD, whichever backend the card is mining on. A card whose driver
exposes no sensor reports 0 — set your own fan curve there.

AMD rigs, and Intel-only rigs, without a Vulkan driver: the start script attempts
to install `mesa-vulkan-drivers` once per boot (needs apt + network); if that
fails it prints it on the miner console (not in the miner log) — install the
package yourself and reboot.

On Hive OS there is no `axehub-miner.conf` to edit: wherever this README says
"set X in the conf", put `X = ...` in the flight sheet's **Extra config
arguments** box instead.

**Tuning on Hive OS.** An NVIDIA card gets a locked core clock, an AMD card a power
cap (saved to `axehub-miner.conf.power`, or `.bc3hash.power` for bc3hash); on a CPU
device the tuner says so and exits without touching the config. It needs root, which
a Hive Shell already is. Stop the miner first — the tuner takes the same GPU as
the running one. From a Hive Shell:

```
miner stop
cd /hive/miners/custom/axehub-miner
./axehub-miner --algo <your algo> --tune hashrate
miner start
```

The `cd` matters: the miner resolves its config relative to the working
directory. The chosen clock is written to its own file next to the config —
`axehub-miner.conf.clock` — and re-applied at every start; the flight sheet
rebuilds the config and never touches it, so there is nothing to copy into the
panel. Your config is not modified by the tuner at all.

The value is an **absolute locked core clock** (what `nvidia-smi -lgc` sets), not
a `+/-` offset — if you prefer to set it in Hive's OC tab, it belongs in **Lock
Core Clock (MHz)**, not **Core Clock**. To clear a saved clock, delete that file
**and** run `nvidia-smi -i <pci-bus-id> -rgc` and then, as a separate command,
`nvidia-smi -i <pci-bus-id> -rmc` (the tuner may also have locked the memory clock) —
a lock outlives the process that set it, so deleting the file alone stops the clock coming back but does not
release a card that is already locked. The miner prints the bus id when it saves;
it is not the same as the miner's device index. A `clock_lock` you set in Extra config wins outright and
the file is ignored, so pin every card in that single line or leave the key out.

### Mining BitcoinIII (BC3) instead

BC3's proof of work is bc3hash (its pre-fork `sha3-256t` is still accepted). Switch
algorithm and pool, and use a BitcoinIII (`bc1...`) address:

```
--algo bc3hash --pool pool.axehub.app:3338 --wallet bc1...
```

or set `algo = bc3hash`, `pool = pool.axehub.app:3338` and a `bc1...` wallet in
`axehub-miner.conf`. Runs on NVIDIA (CUDA) and AMD/other discrete GPUs (OpenCL) — on
Linux, AMD cards use the `linux-x64-amd` package; the 1% dev fee applies the same way.

### Mining poscan

poscan is a **GPU-only** algorithm — NVIDIA (CUDA) or AMD (OpenCL on Linux); it has no CPU
or Vulkan form. There is no AxeHub poscan pool, so point `--pool` at a poscan pool and use
that pool's address format:

```
--algo poscan --pool host:port --wallet YOUR_ADDRESS
```

On NVIDIA nothing extra is needed. On an AMD card take the `linux-x64-amd` package and start
with `run.sh` (as root, or as a member of the `render` group); it sets up the bundled OpenCL
runtime for poscan automatically. On Hive OS set the **Hash algorithm** field to `poscan`.
The 1% dev fee applies the same way.

### Mining rgrpow01

rgrpow01 is a **GPU-only** algorithm — NVIDIA (CUDA) or AMD (OpenCL, with Vulkan as a
fallback). There is no AxeHub rgrpow01 pool, so point `--pool` at an rgrpow01 pool and use
that pool's address format:

```
--algo rgrpow01 --pool host:port --wallet YOUR_ADDRESS
```

On NVIDIA nothing extra is needed. On an AMD card take the `linux-x64-amd` package and start
with `run.sh` (as root, or as a member of the `render` group); it routes rgrpow01 through the
bundled OpenCL runtime (Vulkan as a fallback). On Hive OS set the **Hash algorithm** field to
`rgrpow01`. The 1% dev fee applies the same way.

### Mining on the CPU

The CPU is a first-class backend (any x86-64). Turn it on in any of these ways:

```
--backend cpu          # CPU only, no GPU
--backend auto,cpu     # GPU and CPU at the same time
--cpu                  # shorthand: add the CPU on top of the default GPU backends
--cpu-threads N        # worker threads (omit or 0 = auto, leaves one core free)
```

or in `axehub-miner.conf` set `backend = cpu` (or `auto,cpu`), or
`cpu_threads = auto` (or a number). CPU mining honors `--algo` for whirlpool, bc3hash,
sha3-256t and qpow (poscan and rgrpow01 are GPU-only); bc3hash on the CPU
runs on any processor (AVX2 makes it faster). It is much slower than a GPU —
use it on machines without one, or to mine on spare cores alongside the GPU.

## Verify the build

```sh
axehub-miner --selftest                  # CapStash (Whirlpool)
axehub-miner --algo bc3hash --selftest   # BitcoinIII (bc3hash)
axehub-miner --algo sha3-256t --selftest # BitcoinIII (pre-fork)
axehub-miner --algo qpow --selftest      # qpow
axehub-miner --algo poscan --selftest    # poscan
axehub-miner --algo rgrpow01 --selftest  # rgrpow01
```
Runs a bit-exact GPU==CPU correctness check against a known test vector. It must
print `self-test PASS`.

## Supported hardware

- **NVIDIA (CUDA):** GTX 10xx (Pascal), GTX 16xx / RTX 20xx (Turing),
  RTX 30xx (Ampere), RTX 40xx (Ada), RTX 50xx (Blackwell). On Linux keep the bundled `libcap_cuda.so` and `libvulkan.so.1`
  next to the binary — both load from the miner's own directory.
- **AMD / Intel / other:** any Vulkan-capable GPU (`--backend vulkan`). The
  Vulkan loader ships with the miner; you still need your GPU vendor's Vulkan
  driver.
- **CPU:** any x86-64.
- `--backend auto`: NVIDIA → CUDA, everything else → Vulkan (OpenCL for bc3hash, poscan and rgrpow01).
- On Linux, integrated GPUs (iGPU/APU) are skipped by default when `--backend auto` or
  `vulkan` picks devices. Add `--vulkan-igpu` (or `vulkan_igpu = 1` in the config) to
  mine on them too.
- **OS:** Windows x64, Linux x86-64 (glibc 2.29+), Hive OS (custom miner package). AMD
  cards mining bc3hash, qpow or poscan through the bundled OpenCL runtime on Linux or
  Hive OS need glibc 2.35 or newer (Ubuntu 22.04 or newer; on Hive OS an image based on it).

## Key flags (override the conf)

```
--config FILE             use this config instead of axehub-miner.conf
--pool host:port          the pool to mine on
--wallet ADDR             your payout address
--pass P                  the pool password (d=1: fixed difficulty 1, whirlpool only)
--worker NAME             names this rig to the pool
--fallback h:p[,h:p...]   backup pools, same algorithm; fails back to the first
--algo NAME               whirlpool | bc3hash | sha3-256t | qpow | poscan | rgrpow01
--backend LIST            auto | cuda | opencl | vulkan | cpu, comma-separated
                          (e.g. auto,cpu)
--gpus 0,1                use only these device indices (default: all)
--cpu                     add the CPU as a mining device (see "Mining on the CPU")
--cpu-threads N           CPU worker threads (0 or omitted = auto)
--tune MODE               find + save the best operating point for MODE, then exit
                          (NVIDIA: core + memory clock; AMD on Linux: power cap;
                          admin on Windows, root on Linux and Hive OS)
                          MODE = hashrate | efficiency | balanced | profit | performance
                          (profit needs --kwh + --rev; performance is NVIDIA-only: a safe
                          automatic core-clock overclock, each step verified bit-exact)
--no-tui                  plain scrolling log (headless / service use)
--selftest                bit-exact correctness gate
--list-algos              print the algorithms and their backends, then exit
--list-devices            print the devices the miner sees, then exit
--version                 print the version, then exit
--temp C                  pause a device above C, resume 5 below
--shutdown-temperature C  power the host off if a device reaches C (off by default)
--watchdog-timeout S      restart if a device makes no progress for S seconds
                          (off by default)
--log-file PATH           also append the log here (rotated)
--report [PATH]           write a diagnostic bundle with the wallet and password hidden
--http PORT               status page + API, localhost only
                          (/1/summary, /2/summary, /metrics)
--http-bind lan           also serve on the LAN (e.g. the phone app); anyone on the
                          network can then read the status, including the wallet
                          and worker name
--http-token SEC          required before the API or the status page may change
                          anything (pool, wallet, worker, pause, temperature limit,
                          tune, save, restart); without it those are refused,
                          read-only status stays open
```

## Dev fee

This build mines **1%** of the time to the developer's address (disclosed, and
announced in the log) on whirlpool, bc3hash, qpow, poscan and rgrpow01. The other 99% goes
entirely to your configured wallet.

The optional **Monero (XMR) dual-mine carries no dev fee (0%)**.

## Monero (XMR) dual-mine (optional)

Idle CPU cores can mine **Monero (RandomX)** at the same time the GPU mines
CAP or BC3 — free extra revenue at **0% dev fee** (100% of the XMR goes to your
wallet). Off by default; enable it by setting `xmr_pool` + `xmr_wallet` in
`axehub-miner.conf` (or `--xmr-pool` / `--xmr-wallet`). Needs ~2 GB RAM for the
RandomX dataset.

## Antivirus note

Like every crypto miner, this binary may be flagged by heuristic/ML antivirus
engines — it is not malware. If your AV quarantines it, add a folder exception.
The build is code-protected; please do not repackage it.

## License

Proprietary — **all rights reserved**. The binaries are provided under the
end-user terms in [`LICENSE.txt`](LICENSE.txt) (binary-only; no reverse
engineering, modification, or redistribution). Third-party components and their
licenses are listed in [`THIRD-PARTY-NOTICES.txt`](THIRD-PARTY-NOTICES.txt); the packages
with the AMD runtime also carry its license texts in `THIRD-PARTY-LICENSES-amd.txt`.
