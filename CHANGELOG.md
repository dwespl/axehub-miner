# Changelog

All notable changes to AxeHub Miner are documented here.

## [1.7] - 2026-10-19

Adds bc3hash — BitcoinIII's current proof of work — on NVIDIA and AMD GPUs, and brings AMD
cards to it on Hive OS. One binary — Windows, Linux and Hive OS.

### Added
- **bc3hash (`--algo bc3hash`)** on NVIDIA (CUDA) and AMD or other discrete GPUs (OpenCL).
  BitcoinIII adopted it at its hard fork; the pre-fork `sha3-256t` stays selectable.
- **AMD cards mine bc3hash on Hive OS** out of the box — no setup, and NVIDIA and the other
  algorithms are unaffected.
- **`power_cap`** — on AMD (Linux), `--tune` picks and saves a per-GPU power cap.

### Changed
- Packaging and reliability improvements across all three builds.

## [1.6] - 2026-08-18

An OpenCL backend that makes AMD cards faster, and a tuner that now also tunes the GPU
memory clock. Same two coins (CAP, BC3), one binary — Windows, Linux and Hive OS.

### Added
- **OpenCL compute backend for AMD.** `--backend opencl` (or `cuda,opencl` on a mixed
  NVIDIA+AMD rig) drives AMD GPUs through OpenCL, which is faster than Vulkan on AMD, using
  the OpenCL runtime already on the system. NVIDIA keeps running CUDA by default; add
  `opencl` only for the AMD card.
- **Memory-clock tuning in `--tune`.** It measures on your card whether a lower memory
  clock saves power without costing hashrate, pins it when it pays off, and reports what it
  saved. New `mem_lock` config key, same grammar as `clock_lock`. NVIDIA Ampere or newer
  only; any other card is told plainly and keeps mining.
- **Board-power reporting for AMD cards on Linux**, from the same kernel sensor node as
  temperature and fan, so watts and efficiency show on a Linux AMD rig.

### Changed
- The core clock is tuned with the memory clock already set — they are one operating point,
  so re-run `--tune` after upgrading.
- Clearer output while the tuner runs, and smoother behaviour on long unattended runs.

## [1.5] - 2026-08-16

Temperature and fan monitoring for AMD cards, alongside NVIDIA. Same two coins (CAP,
BC3), same three backends (CUDA / Vulkan / CPU), one binary — Windows, Linux and Hive OS.

### Added
- **Temperature and fan readings for AMD GPUs.** The miner reads AMD's control library on
  Windows and the kernel's sensor nodes on Linux, matching each card by its PCI bus id; on
  Linux, any card whose driver exposes a sensor node is read the same way. The readings
  appear in the status display, in the HTTP API and on the Hive OS panel, and `temp_limit`
  acts on them.
- A card whose sensors are unavailable reports no value, and the thermal limit stays out
  of the way on that device.
- Every CPU kernel proves itself against the reference implementation on your own machine
  before it is used to mine.

### Changed
- `--selftest` on the CPU device checks every CPU kernel that could be selected on your
  machine.
- The CPU backend's Whirlpool starting kernel is an 8-wide batch; per-machine autotuning
  has the final say.
- The status display's device column shows the card model, with the vendor and brand words
  dropped so the model itself has the room.

## [1.4] - 2026-08-13

Clock tuning now sticks on Hive OS, and the CPU backend is more careful about which of
its kernels it picks. Same two coins (CAP, BC3), same three backends (CUDA / Vulkan /
CPU), one binary — Windows, Linux and Hive OS.

### Fixed
- **On Hive OS, a clock chosen by `--tune` no longer disappears.** It is saved beside
  the config in its own file, which applying a flight sheet does not touch, so the
  miner re-applies it at every start with nothing to copy into the panel by hand.

### Changed
- **`--tune` keeps its pick in its own file, `axehub-miner.conf.clock`, beside the
  config.** The config itself is left alone. To clear a saved clock, delete that file;
  `nvidia-smi -i <pci-bus-id> -rgc` releases the lock on a card that is already running
  (the miner prints the bus id when it saves). If you upgraded from
  an earlier version and have a `clock_lock` line in your config, that line still wins —
  remove it to let tuning take over.
- A `clock_lock` line you set yourself in the flight sheet's **Extra config arguments**
  box wins outright, and the saved file is ignored. Pin every card you want in that
  single line, or leave the key out of the box and let tuning own it.

### Added
- The Hive OS package now ships its `README.md`, covering the flight-sheet fields and
  tuning end to end: running it from a Hive Shell, which field in the OC tab takes an
  absolute locked clock rather than an offset, and how to clear one.
- The CPU backend now takes the median of several measurements per kernel on both
  algorithms, and sizes each measurement to the machine so the check costs about the
  same whether the CPU is fast or slow. On a machine where two kernels are close, the
  faster one is picked more reliably.

## [1.3] - 2026-08-11

Adds **Hive OS** support and native Blackwell (RTX 50xx) GPU code. Same two coins
(CAP, BC3), same three backends (CUDA / Vulkan / CPU), one binary — Windows, Linux
and Hive OS. All three packages ship the Vulkan loader (`vulkan-1.dll` /
`libvulkan.so.1`) beside the binary — your GPU vendor's Vulkan driver is still
required — and the Linux and Hive OS builds run on glibc 2.31 or newer.

### Added
- **Hive OS package** — `axehub-miner-1.3.tar.gz`. Paste the asset URL as the
  custom miner **Installation URL**, fill in the flight sheet and mine.
  Hashrate, accepted/rejected shares and uptime report to the Hive OS dashboard;
  per-GPU temperature and fan are reported on the CUDA (NVIDIA) path, and read 0
  with no card mapping on the Vulkan path.
- Flight-sheet support for the algorithm via **Hash algorithm** (`whirlpool` or
  `sha3-256t`; coin names accepted as aliases), for a fallback pool (a second
  `host:port` in the Pool URL field), and for tuning keys such as
  `backend = vulkan` in the extra config box.
- **Native Blackwell (RTX 50xx) GPU code** — the supported NVIDIA range spans
  GTX 10xx through RTX 50xx and newer.
- On an AMD rig, or an Intel-only rig, whose Hive OS image ships no Vulkan driver,
  the start script attempts to install `mesa-vulkan-drivers` once per boot (needs
  working apt and network); if it fails it prints that on the miner console and the
  Vulkan backend stays unavailable.

## [1.2] - 2026-08-09

Tuning, monitoring and reliability improvements. Same two coins (CAP, BC3),
same three backends (CUDA / Vulkan / CPU), one binary, Windows and Linux.

### Added
- **`--tune MODE`** picks what the clock tuner optimizes for:
  `hashrate` | `efficiency` | `balanced` | `profit` (profit uses `--kwh` and
  `--rev`). It sweeps on the algorithm you selected, saves the chosen clock to the
  conf, and re-applies it at startup. (`--tune-eff` is a shorthand for `--tune efficiency`.)
- **`--http-bind` and `--http-token` command-line flags** — `--http-bind lan`
  serves the status page on the LAN (e.g. the phone app); `--http-token SECRET`
  gates remote pool/wallet changes over the API. Localhost only unless you pass `lan`.

### Changed
- Smoother behavior on long unattended runs and across pool reconnects.

## [1.1] - 2026-07-13

Adds a second coin: **BitcoinIII (BC3)**, triple-SHA3-256. One binary now mines
either CapStash (CAP) or BitcoinIII (BC3), on CUDA / Vulkan / CPU, on Windows and
Linux.

### Added
- **BitcoinIII (BC3) support** — `--algo sha3-256t` (aliases: `bc3`, `sha3`).
  Point at `pool.axehub.app:3338` with a `bc1...` wallet. Bit-exact GPU==CPU
  self-test against the SHA3 NIST vector: `--algo sha3-256t --selftest`.
- Optional BC3 block and an `algo` key in `axehub-miner.conf`; `--algo` is
  documented in `--help` and the README.

### Changed
- The 1% disclosed dev fee applies to BC3 the same way as to CAP. The optional
  Monero (XMR) dual-mine remains 0% fee.
- An unrecognized `--algo` value now exits with a clear error instead of silently
  defaulting (which could mine the wrong coin on a mismatched pool).

## [1.0] - 2026-06-14

First public release.

### Added
- Multi-backend solo miner for **CapStash (CAP)** — Whirlpool-512 fold PoW.
- Backends from one binary: CUDA (NVIDIA), Vulkan (AMD / Intel & others), CPU
  (AVX2 autotune).
- Live TUI + HTTP status page, bit-exact GPU==CPU self-test, automatic efficiency
  tuner (best MH/W).
- Optional Monero (XMR) CPU dual-mine at 0% dev fee.
- 1% disclosed dev fee. Windows and Linux builds.
