# RCPU Miner

CPU miner for the **RCPU** chain — **RandomX** (active) and **Etchash** (from block height 21000).

Download the latest release: **[v1.0.0](https://github.com/btcrcpu-max/rcpuminer/releases/tag/v1.0.0)**

## Downloads

| File | Platform |
|---|---|
| `rcpuminer-1.0.0-win-x64.zip` | Windows 10/11 x64 — CLI miner + GUI launcher |
| `rcpuminer-1.0.0-linux-x64.tar.gz` | Linux x86_64 — CLI miner + launcher scripts |
| `rcpuminer-1.0.0-src.tar.gz` | Full source tree (GPL compliance) |
| `sha256sums.txt` | Checksums |

## Mining modes

| Mode | Port | Algorithm | Fee |
|---|---|---|---|
| RandomX pool | 3334 (also 3333 / 8080) | RandomX | 0.5% PPLNS |
| RandomX Solo | 3335 | RandomX | 0% — full block reward to the finder |
| Etchash pool | 8888 | Etchash | 0.5% PPLNS (from height 21000) |
| Etchash Solo | 8889 | Etchash | 0% (from height 21000) |

Pool host: `stratum+tcp://207.57.129.188:<port>`
Username: `YOUR_WALLET.worker1` — the username must be **your own** wallet address, rewards are
credited to it.

## Quick start

### Windows

1. Extract the zip.
2. Run `rcpuminer-gui.exe`, type your wallet address, pick a mode, press **Start mining**.
   Or run `start-rcpuminer.bat` in a console.

### Linux

```bash
tar xzf rcpuminer-1.0.0-linux-x64.tar.gz
cd .
./start-rcpuminer.sh              # interactive
./start.sh <wallet> 3334 2        # non-interactive
```

### Command line

```
rcpuminer -a randomx|etchash -o stratum+tcp://207.57.129.188:<port> \
          -u <wallet>.worker1 -p x -t <threads>
```

## Requirements

- **RandomX**: ~2 GB RAM for full-memory mode. When less is available (or `RCPU_LIGHT=1` is
  set) the miner automatically runs in light mode (~256 MB, slower but identical hashes).
- **Etchash**: light on memory, but CPU Etchash is slow by design.

## Implementation notes

- 112-byte block header = 80 legacy bytes + 32-byte `hashRandomX` (zeroed while hashing)
- RandomX salt `RandomX-RCPU\x01`; difficulty is checked against
  `randomx_calculate_commitment()` and the hash is submitted in the trailing field
- Etchash sets version bit 30, uses `keccak256(header112)` as the header hash and
  `hashimotoLight` for the loop; the result is stored in the `hashRandomX` field

## Building from source

See [BUILD.md](BUILD.md) in the source archive. It lists the four patches that make an upstream
`cpuminer-scash` build RCPU-compatible (RandomX salt, difficulty scaling, epoch seed, prevhash
word order) and the exact Linux/Windows compile commands.

## Verified

- Live shares on the official pool (`207.57.129.188:3334/3335`): `accepted: 36/36 (100.00%)`
  over a 12-minute run with zero rejects and zero disconnects
- Etchash path validated against a Miningcore instance: share accepted, submitted block carried
  `version = 0x60000000` and a matching `hashRandomX` field
- Hash output cross-checked byte-for-byte against independent recomputation using the same
  RandomX / Etchash libraries the node uses

## Known limitation

The Windows binaries were cross-compiled on Linux and were **not** executed on a Windows machine
during development. They are valid PE32+ images depending only on system DLLs — please open an
issue if anything behaves unexpectedly.

## License

Upstream cpuminer-scash is GPL-2.0; RandomX is BSD-3-Clause; the Etchash/keccak code originates
from the RCPU node (MIT). Binary releases are accompanied by the full source tree.
