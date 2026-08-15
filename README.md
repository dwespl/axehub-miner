# AxeHub Miner

A fast multi-backend **solo miner** for two proof-of-work coins from one binary:

- **CapStash (CAP)** — Whirlpool-512 fold (default)
- **BitcoinIII (BC3)** — triple-SHA3-256 (`--algo sha3-256t`)

It mines on NVIDIA (CUDA), AMD/Intel & other GPUs (Vulkan), or CPU
(x86-64, vector-accelerated), on **Windows, Linux and Hive OS**. Live TUI + HTTP status, bit-exact GPU==CPU self-test,
a built-in clock tuner, 1% disclosed dev fee.

There is no pool hardcoded anywhere: point `--pool` at any pool serving these
coins. Ours are `pool.axehub.app:3336` (CAP) and `pool.axehub.app:3338` (BC3),
and they are what the examples below use.

> Distributed in **binary form only**. See [Releases](../../releases) for the
> latest build.

## Download

Grab the archive for your OS from the [latest release](../../releases/latest):

| OS | Archive | Backends |
|----|---------|----------|
| Windows x64 | `axehub-miner-v1.4-windows-x64.zip` | CUDA · Vulkan · CPU |
| Linux x64   | `axehub-miner-v1.4-linux-x64.tar.gz` | CUDA · Vulkan · CPU |
| Hive OS     | `axehub-miner-1.4.tar.gz` | CUDA · Vulkan · CPU |

## Quick start

**Windows**
1. Extract the zip.
2. Open `axehub-miner.conf`, set `wallet` to your CapStash (CAP) address.
3. Run `axehub-miner.exe`.

**Linux**
```sh
tar -xzf axehub-miner-v1.4-linux-x64.tar.gz
cd axehub-miner-v1.4-linux-x64
chmod +x axehub-miner libcap_cuda.so
# edit axehub-miner.conf -> set your CAP wallet
./axehub-miner
```

A live status panel appears; the pool credits blocks to your address (solo).

**Hive OS**

1. **Wallets → Coins → Add Coin** — add a custom coin (`CapStash` or
   `BitcoinIII`); the algorithm you pick there is only a Hive label, the miner
   takes its algorithm from the **Hash algorithm** field below. Then **Add
   Wallet** for that coin with your `cap1...` / `bc1...` address. The template
   field also accepts a literal address instead of `%WAL%`.
2. Create a Flight Sheet, pick **Custom** as the miner, and set the pool to
   **Configure in miner**.
3. **Installation URL** — the URL of `axehub-miner-1.4.tar.gz` from the
   [latest release](../../releases/latest). **Miner name** — `axehub-miner`.
4. Fill in the flight sheet:

   | Field | Value |
   |---|---|
   | Pool URL (required) | `host:port` — the port is required. `pool.axehub.app:3336` for CAP, `pool.axehub.app:3338` for BC3 |
   | Wallet and worker template (required) | `%WAL%.%WORKER_NAME%` |
   | Hash algorithm | `whirlpool` for CAP (the default if left blank) — **required for BC3**: set `sha3-256t` |
   | Pass | optional — `d=1` asks the pool for fixed difficulty 1 |
   | Extra config arguments | optional tuning keys, one per line |

5. Apply the flight sheet to your rig.

A second `host:port` in the same Pool URL field, separated by a space, is used as
a fallback; it must serve the same coin (a second line is ignored). Extra config
keys are appended last and override earlier values, so keep the pool, wallet and
algorithm in their own fields. `temp_limit` applies to the CUDA path only.

Hashrate, accepted/rejected shares and uptime report to the Hive OS dashboard.
Per-GPU temperature and fan are reported on the CUDA (NVIDIA) path; on the Vulkan
path the miner cannot read those sensors, so they show 0 with no card mapping —
including on an NVIDIA card forced to `backend = vulkan`. Set your own fan curve.

AMD rigs, and Intel-only rigs, without a Vulkan driver: the start script attempts
to install `mesa-vulkan-drivers` once per boot (needs apt + network); if that
fails it prints it on the miner console (not in the miner log) — install the
package yourself and reboot.

On Hive OS there is no `axehub-miner.conf` to edit: wherever this README says
"set X in the conf", put `X = ...` in the flight sheet's **Extra config
arguments** box instead.

**Clock tuning on Hive OS.** NVIDIA only — the tuner drives NVML, and on an AMD or
CPU device it says so and exits without touching the config. It needs root, which
a Hive Shell already is. Stop the miner first — the tuner takes the same GPU as
the running one. From a Hive Shell:

```
miner stop
cd /hive/miners/custom/axehub-miner
./axehub-miner --algo sha3-256t --tune hashrate
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
**and** run `nvidia-smi -i <pci-bus-id> -rgc` — an NVML lock outlives the process
that set it, so deleting the file alone stops the clock coming back but does not
release a card that is already locked. The miner prints the bus id when it saves;
it is not the same as the miner's device index. A `clock_lock` you set in Extra config wins outright and
the file is ignored, so pin every card in that single line or leave the key out.

### Mining BitcoinIII (BC3) instead

BC3 is triple-SHA3-256. Switch algorithm and pool, and use a BitcoinIII
(`bc1...`) address:

```
--algo sha3-256t --pool pool.axehub.app:3338 --wallet bc1...
```

or set `algo = sha3-256t`, `pool = pool.axehub.app:3338` and a `bc1...` wallet in
`axehub-miner.conf`. Same three backends (CUDA / Vulkan / CPU); the 1% dev fee
applies the same way.

### Mining on the CPU

The CPU is a first-class backend (x86-64, AVX2-accelerated with a scalar
fallback). Turn it on in any of these ways:

```
--backend cpu          # CPU only, no GPU
--backend auto,cpu     # GPU and CPU at the same time
--cpu                  # shorthand: add the CPU on top of the default GPU backends
--cpu-threads N        # worker threads (omit or 0 = auto, leaves one core free)
```

or in `axehub-miner.conf` set `backend = cpu` (or `auto,cpu`), or
`cpu_threads = auto` (or a number). CPU mining honors `--algo`, so it works for
both CapStash and BitcoinIII. It is much slower than a GPU — use it on machines
without one, or to mine on spare cores alongside the GPU.

## Verify the build

```sh
axehub-miner --selftest                  # CapStash (Whirlpool)
axehub-miner --algo sha3-256t --selftest # BitcoinIII (triple-SHA3)
```
Runs a bit-exact GPU==CPU correctness check against a known test vector
(Whirlpool NESSIE / SHA3 NIST + 1M nonces). It must print `self-test PASS`.

## Supported hardware

- **NVIDIA (CUDA, fastest):** GTX 10xx (Pascal), GTX 16xx / RTX 20xx (Turing),
  RTX 30xx (Ampere), RTX 40xx (Ada), RTX 50xx (Blackwell — native code), and
  newer via PTX. On Linux keep the bundled `libcap_cuda.so` and `libvulkan.so.1`
  next to the binary — both load from the miner's own directory.
- **AMD / Intel / other:** any Vulkan-capable GPU (`--backend vulkan`). The
  Vulkan loader ships with the miner; you still need your GPU vendor's Vulkan
  driver.
- **CPU:** any x86-64, with runtime AVX2 dispatch and a scalar fallback;
  AVX-512 is used for `sha3-256t` where the CPU supports it.
- `--backend auto`: NVIDIA → CUDA, everything else → Vulkan.
- **OS:** Windows x64, Linux x86-64 (glibc 2.29+), Hive OS (custom miner package).

## Key flags (override the conf)

```
--pool host:port   --wallet ADDR   --pass d=1   --worker NAME
--algo whirlpool|sha3-256t   --backend auto|cuda|vulkan|cpu   --gpus 0,1
--cpu   --cpu-threads N   add the CPU as a mining device (see "Mining on the CPU")
--tune MODE  sweep + lock the best GPU clock for MODE, then exit (NVIDIA; admin on
             Windows, root on Linux and Hive OS;
             MODE = hashrate|efficiency|balanced|profit - profit needs --kwh + --rev)
--no-tui     plain scrolling log (headless / service use)
--selftest   bit-exact correctness gate
--http PORT       status page + API (localhost only)
--http-bind lan   also serve on the LAN (e.g. the phone app)
--http-token SEC  required before the API may change pool or wallet (without it
                  those are refused; read-only status stays open)
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
