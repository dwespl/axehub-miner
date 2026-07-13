# Changelog

All notable changes to AxeHub Miner are documented here.

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
