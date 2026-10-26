# CPU Cache Simulator

A CPU cache simulator with a C++17 simulation engine, a C++ command-line runner, and a Python/Streamlit dashboard. The dashboard calls the C++ engine through a C-compatible interface and `ctypes`.

## Run the dashboard on macOS

Install Xcode Command Line Tools if `clang++` is not available, then from this folder run:

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
streamlit run app.py --server.address 127.0.0.1
```

The Python bridge builds the C++ shared library automatically on first use. The dashboard runs locally at the URL printed in the terminal.

## Features

- One to three cache levels, with independently configurable capacity and mapping at each level.
- Direct-mapped, set-associative, and fully associative mapping.
- LRU, FIFO, and deterministic random replacement.
- Write-back or write-through behavior, with write-allocate or no-write-allocate on write misses.
- Read and write operations, dirty-line writebacks, and memory read/write transaction counts.
- Configurable access latency per cache level and memory latency, with per-access cycle counts and average access time.
- Editable traces or uploaded `.txt`, `.trace`, and `.csv` files. Trace lines accept `R 0x40`, `W 0x80`, or a bare address (a read). Two-column CSV traces accept either `operation,address` or `address,operation`, with an optional header. Decimal and `0x` hexadecimal addresses work; blank lines and `#` comments are ignored.
- Per-access result, set, tag, and cycle count for each cache level, plus per-level statistics and final resident-line state.
- Downloadable CSV output for the per-access results.
- A light-toned dashboard with a cache-flow illustration, hit/miss comparison, access-latency timeline, and address-to-set mapping plot.

All levels use the same block size and are limited to one million lines each. The hierarchy is non-inclusive: evicting a block from one level does not invalidate a copy in another level. Lower-level transaction counts include demand accesses, fills, propagated writes, and dirty writebacks. Random replacement uses a fixed seed, so the same configuration and trace produce repeatable results.

Cycle estimates add the latency of each cache lookup and each memory transaction performed by the simulator. They do not model overlapped requests, queueing, or bandwidth contention. Dirty lines still resident at the end of a trace are shown in the final cache contents and are not counted as memory writes until evicted.

## Run the C++ command-line version

The command-line program is a one-level direct-mapped runner using LRU and write-back/write-allocate defaults:

```sh
clang++ -std=c++17 -Wall -Wextra -pedantic cache.cpp main.cpp -o cache_sim
./cache_sim 256 16
```

Enter one address per line for a read, or prefix it with `R` or `W` (for example `W 0x40`). Press Control-D to finish. The dashboard exposes the full configuration set.

## Model scope

Addresses identify byte locations; each operation touches the containing block. The simulator models demand accesses and block transfers but does not model timing, coherence between CPU cores, instruction versus data caches, spatial sub-block transfers, or a real processor trace format. The hierarchy is kept non-inclusive for a clear and tractable first release.
