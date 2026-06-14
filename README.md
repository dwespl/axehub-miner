# AxeHub Miner

A fast multi-backend **solo miner for CapStash (CAP)** — a Whirlpool-512 fold
proof-of-work coin. One binary mines on NVIDIA (CUDA), AMD/Intel & other GPUs
(Vulkan), or CPU (AVX2), on **Windows and Linux**. Live TUI + HTTP status,
bit-exact GPU==CPU self-test, automatic efficiency tuning, 1% disclosed dev fee.

> Distributed in **binary form only**. See [Releases](../../releases) for the
> latest build.

## Download

Grab the archive for your OS from the [latest release](../../releases/latest):

| OS | Archive | Backends |
|----|---------|----------|
| Windows x64 | `axehub-miner-v1.0-windows-x64.zip` | CUDA · Vulkan · CPU |
| Linux x64   | `axehub-miner-v1.0-linux-x64.tar.gz` | CUDA · Vulkan · CPU |

## Quick start

**Windows**
1. Extract the zip.
2. Open `axehub-miner.conf`, set `wallet` to your CapStash (CAP) address.
3. Run `axehub-miner.exe`.

**Linux**
```sh
tar -xzf axehub-miner-v1.0-linux-x64.tar.gz
cd axehub-miner-v1.0-linux-x64
chmod +x axehub-miner libcap_cuda.so
# edit axehub-miner.conf -> set your CAP wallet
./axehub-miner
```

A live status panel appears; the pool credits blocks to your address (solo).

## Verify the build

```sh
axehub-miner --selftest      # ./axehub-miner --selftest on Linux
```
Runs a bit-exact GPU==CPU correctness check (Whirlpool NESSIE vector + 1M
nonces). It must print `self-test PASS`.

## Supported hardware

- **NVIDIA (CUDA, fastest):** GTX 10xx (Pascal), GTX 16xx / RTX 20xx (Turing),
  RTX 30xx (Ampere), RTX 40xx (Ada) and newer (via PTX). On Linux keep the
  bundled `libcap_cuda.so` next to the binary — it auto-loads.
- **AMD / Intel / other:** any Vulkan-capable GPU (`--backend vulkan`).
- **CPU:** x86-64 with runtime AVX2 dispatch (scalar fallback on non-AVX2).
- `--backend auto`: NVIDIA → CUDA, everything else → Vulkan.

## Key flags (override the conf)

```
--pool host:port   --wallet cap1...   --pass d=1   --worker NAME
--backend auto|cuda|vulkan   --gpus 0,1   --cpu
--tune-eff   sweep + lock the efficiency-optimal GPU core clock (NVIDIA)
--no-tui     plain scrolling log (headless / service use)
--selftest   bit-exact correctness gate
```

## Dev fee

This build mines **1%** of the time to the developer's address (disclosed;
~6 s per 600 s). The other 99% goes entirely to your configured wallet.

The optional **Monero (XMR) dual-mine carries no dev fee (0%)** — the 1% applies
to CapStash (CAP) mining only.

## Monero (XMR) dual-mine (optional)

Idle CPU cores can mine **Monero (RandomX)** at the same time the GPU mines
CapStash — free extra revenue at **0% dev fee** (100% of the XMR goes to your
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
licenses are listed in [`THIRD-PARTY-NOTICES.txt`](THIRD-PARTY-NOTICES.txt).
