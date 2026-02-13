# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Xapiand is a distributed RESTful search and storage engine built in C++17. It accepts JSON/MessagePack documents, indexes them via the Xapian library, and serves queries over HTTP (port 8880) and a binary protocol (port 8890). It supports clustering via RAFT consensus and node discovery via UDP multicast.

## Build Commands

```bash
# Configure (out-of-source build)
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug

# Build
cmake --build . -j$(nproc)

# Build with tests enabled
cmake .. -DBUILD_TESTS=ON
cmake --build . -j$(nproc)

# Run all tests
ctest

# Run a single test
./bin/xapiand_test_<name>   # e.g. ./bin/xapiand_test_serialise

# Build with benchmarks
cmake .. -DBUILD_BENCHMARKS=ON
```

Tests require Google Test (`GTest`). Benchmarks require Google Benchmark.

### Key CMake Options

| Option | Default | Description |
|---|---|---|
| `BUILD_TESTS` | OFF | Build test suite |
| `BUILD_BENCHMARKS` | OFF | Build benchmarks |
| `CLUSTERING` | OFF | Enable distributed RAFT clustering |
| `DATABASE_WAL` | ON | Write-ahead logging |
| `DATA_STORAGE` | ON | Binary data storage layer |
| `CHAISCRIPT` | ON | ChaiScript scripting engine |
| `V8` | OFF | V8 JavaScript engine (alternative to ChaiScript) |

## Architecture

### Server Layer (`src/servers/`)

Event-driven async architecture using libev (`src/ev/`):

- **XapiandManager** (`manager.h`) — Top-level coordinator, manages server lifecycle
- **XapiandServer** (`server.h`) — Multiplexes HTTP and binary protocol servers
- **HttpServer / ClientHttp** (`server_http.h`, `client_http.h`) — REST API handling
- **BinaryServer / ClientBinary** (`server_binary.h`, `client_binary.h`) — Binary protocol
- **DiscoveryServer** (`server_discovery.h`) — UDP multicast node discovery
- **RaftServer** (`server_raft.h`) — RAFT consensus for clustering

All servers and clients inherit from **Worker** (`worker.h`), which provides a hierarchical parent-child lifecycle and event loop integration.

### Database Layer

- **DatabaseHandler** (`database_handler.h`) — Routes requests to databases, handles indexing/querying
- **Database** (`database.h`) — Wraps Xapian::Database with connection pooling (MAX_DATABASES=400)
- **StorageLayer** — LZ4-compressed binary storage with WAL support (4KB block alignment)

### Query Processing

- **QueryDSL** (`query_dsl.h`) — Parses JSON query DSL into Xapian queries. Supports `_and`, `_or`, `_not`, `_filter`, `_range`, `_in`, wildcards, phrases, geospatial queries
- **Schema** (`schema.h`) — Field type system with index strategies (terms, values, partials). Manages type validation, stemming, stop-words, UUID encoding. This is the largest source file (~312KB)
- **Serialise** (`serialise.h`) — Type-specific serializers for float, integer, date, time, geo, UUID, etc.

### Subsystems

- **Geospatial** (`src/geospatial/`) — HTM (Hierarchical Triangular Mesh) spatial indexing, EWKT parsing, polygon/circle geometry
- **Aggregations** (`src/multivalue/`) — Bucket and metric aggregations over Xapian result sets
- **MsgPack** (`src/msgpack/`) — MessagePack serialization with JSON interconversion
- **Metrics** (`src/prometheus/`) — Prometheus counters, gauges, histograms
- **Scripting** (`src/chaiscript/`, `script.h`) — ChaiScript (default) or V8 scripting with LRU cache

### Bundled Libraries

Located directly under `src/`: `rapidjson`, `lz4`, `fmt`, `cppcodec`, `cuuid`, `sparsehash`, `tclap` (CLI args), `v8pp`.

## Testing

- **`tests/`** — Modern GTest-based tests (currently `test_string.cc`)
- **`oldtests/`** — Comprehensive legacy test suite covering: boolparser, compressor, endpoint, fieldparser, geospatial, geospatial_query, uuid, hash, lru, msgpack, patcher, phonetic, query, queue, serialise, serialise_list, sort, storage, string_metric, threadpool, url_parser, wal
- Each old test has `test_<name>.cc` (main) and `set_<name>_test.cc` (test cases)
- Test fixtures are in `oldtests/` subdirectories

## Docker

Multi-stage Docker build in `contrib/docker/xapiand/Dockerfile` — builds custom Xapian from source, then creates an Alpine-based runtime image exposing port 8880.

## Key Defaults (`src/xapiand.h`)

- HTTP port: 8880, Binary port: 8890
- Discovery: multicast 224.2.2.88:58870, RAFT: 224.2.2.89:58880
- Max clients: 1000, Max databases: 400, DB pool: 300
- Server threads: 10, Flush threshold: 100K docs
