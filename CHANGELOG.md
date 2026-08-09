# Changelog

All notable changes to AxeHub Miner are documented here.

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
