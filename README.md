# Low-Level Storage Engine & Concurrent B+Tree Index

A high-performance, disk-backed, thread-safe storage engine and B+Tree index built from scratch in C++20. Designed for low latency, hardware alignment, cache efficiency, and fine-grained index concurrency control (latch crabbing).

## Architectural Roadmap

- **Layer 1: Storage & Page Contract**
  - Fixed 4096-byte hardware/OS page alignment
  - `DiskManager`: File-backed block allocation and synchronous/direct page I/O
- **Layer 2: Buffer Pool Management**
  - Frame table and Page table (`page_id_t` -> `frame_id_t`)
  - Eviction policy (LRU / Clock Replacer)
  - Pinning lifecycle & dirty tracking
- **Layer 3: Page Layouts & B+Tree Nodes**
  - Flexible array layout for internal and leaf nodes
  - Cache-line optimization and zero-copy byte casting
- **Layer 4: Single-Threaded Index Operations**
  - Point queries (`Find`)
  - Node splitting on insert
  - Node merging / key stealing (borrowing) on delete
  - STL-compliant forward iterator for range scans
- **Layer 5: Concurrency Control**
  - Reader-Writer page latches (`std::shared_mutex`)
  - Latch crabbing / lock coupling protocols (pessimistic and optimistic)

## Directory Structure

```
storage-engine/
├── CMakeLists.txt          # Modern C++20 build configuration
├── src/
│   ├── include/            # Public header files
│   │   ├── common/         # Global definitions, constants, types
│   │   ├── storage/
│   │   │   ├── disk/       # DiskManager headers
│   │   │   ├── page/       # Page, Header, and BPlusTreePage layouts
│   │   │   ├── buffer/     # BufferPoolManager and Replacer headers
│   │   │   └── index/      # BPlusTree and Iterator headers
│   │   └── concurrency/    # Latches and synchronization guards
│   ├── storage/            # Implementation files (.cpp)
│   └── concurrency/
└── test/                   # Unit tests & concurrency stress tests
```

## Building & Testing

Requires a C++20 compatible compiler (`g++ >= 11` or `clang++ >= 13`) and CMake (`>= 3.20`).

```bash
mkdir build && cd build
cmake ..
make -j$(nproc)
ctest --output-on-failure
```
