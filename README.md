# AxeHub Miner

A fast multi-backend **solo miner** for two proof-of-work coins from one binary:

- **CapStash (CAP)** — Whirlpool-512 fold (default)
- **BitcoinIII (BC3)** — triple-SHA3-256 (`--algo sha3-256t`)

It mines on NVIDIA (CUDA), AMD/Intel & other GPUs (Vulkan), or CPU (AVX2), on
**Windows and Linux**. Live TUI + HTTP status, bit-exact GPU==CPU self-test,
automatic efficiency tuning, 1% disclosed dev fee.

> Distributed in **binary form only**. See [Releases](../../releases) for the
> latest build.

## Download

Grab the archive for your OS from the [latest release](../../releases/latest):

| OS | Archive | Backends |
|----|---------|----------|
| Windows x64 | `axehub-miner-v1.1-windows-x64.zip` | CUDA · Vulkan · CPU |
| Linux x64   | `axehub-miner-v1.1-linux-x64.tar.gz` | CUDA · Vulkan · CPU |

## Quick start

**Windows**
1. Extract the zip.
2. Open `axehub-miner.conf`, set `wallet` to your CapStash (CAP) address.
3. Run `axehub-miner.exe`.

**Linux**
```sh
tar -xzf axehub-miner-v1.1-linux-x64.tar.gz
cd axehub-miner-v1.1-linux-x64
chmod +x axehub-miner libcap_cuda.so
# edit axehub-miner.conf -> set your CAP wallet
./axehub-miner
```

A live status panel appears; the pool credits blocks to your address (solo).

### Mining BitcoinIII (BC3) instead

BC3 is triple-SHA3-256. Switch algorithm and pool, and use a BitcoinIII
(`bc1...`) address:

```
--algo sha3-256t --pool pool.axehub.app:3338 --wallet bc1...
```

or set `algo = sha3-256t`, `pool = pool.axehub.app:3338` and a `bc1...` wallet in
`axehub-miner.conf`. Same three backends (CUDA / Vulkan / CPU); the 1% dev fee
applies the same way.

## Verify the build

```sh
axehub-miner --selftest                  # CapStash (Whirlpool)
axehub-miner --algo sha3-256t --selftest # BitcoinIII (triple-SHA3)
```
Runs a bit-exact GPU==CPU correctness check against a known test vector
(Whirlpool NESSIE / SHA3 NIST + 1M nonces). It must print `self-test PASS`.

## Supported hardware

- **NVIDIA (CUDA, fastest):** GTX 10xx (Pascal), GTX 16xx / RTX 20xx (Turing),
  RTX 30xx (Ampere), RTX 40xx (Ada) and newer (via PTX). On Linux keep the
  bundled `libcap_cuda.so` next to the binary — it auto-loads.
- **AMD / Intel / other:** any Vulkan-capable GPU (`--backend vulkan`).
- **CPU:** x86-64 with runtime AVX2 dispatch (scalar fallback on non-AVX2).
- `--backend auto`: NVIDIA → CUDA, everything else → Vulkan.

## Key flags (override the conf)

```
--pool host:port   --wallet ADDR   --pass d=1   --worker NAME
--algo whirlpool|sha3-256t   --backend auto|cuda|vulkan   --gpus 0,1   --cpu
--tune-eff   sweep + lock the efficiency-optimal GPU core clock (NVIDIA)
--no-tui     plain scrolling log (headless / service use)
--selftest   bit-exact correctness gate
```

## Dev fee

This build mines **1%** of the time to the developer's address (disclosed;
~6 s per 600 s), for CapStash (CAP) and BitcoinIII (BC3) alike. The other 99%
goes entirely to your configured wallet.

The optional **Monero (XMR) dual-mine carries no dev fee (0%)** — the 1% applies
to CapStash (CAP) / BitcoinIII (BC3) mining only.

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
licenses are listed in [`THIRD-PARTY-NOTICES.txt`](THIRD-PARTY-NOTICES.txt).
