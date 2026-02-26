# Xapiand

## A RESTful Search Engine

Xapiand is *A Modern Highly Available Distributed RESTful Search and Storage
Engine built for the Cloud and with Data Locality in mind*. It takes JSON
(or MessagePack) documents and indexes them efficiently for later retrieval.

Official site is at [https://kronuz.io/Xapiand](https://kronuz.io/Xapiand)


## Features

- **RESTful JSON API** — Document oriented, no upfront schema definition required
- **Search Engine** — Built on top of [Xapian](https://xapian.org) with (Near) Real-Time Search
- **Storage Engine** — File storage modeled after Facebook's Haystack
- **Aggregations** — Bucket and metric aggregations over result sets
- **Multi-Tenant / Multi-Types** — Multiple indexes with independent schema configuration per type
- **Geo-spatial Support** — Hierarchical Triangular Mesh (HTM) indexing, multiple Coordinate Reference Systems (including WGS84), EWKT
- **High Availability** — Automatic node operation rerouting, replicas for data locality, read/search on any replica, reliable asynchronous replication
- **Clustering** — Distributed via RAFT consensus with node auto-discovery (UDP multicast)
- **Partitioning** — Social-Based (SPAR) and Random Consistent partitioning and replication strategies
- **Scripting** — ChaiScript (default) and V8 JavaScript engine support
- **Event-driven** — Asynchronous architecture using libev, written in modern C++17


## Quick Start

### Install with Homebrew

```sh
brew tap Kronuz/tap
brew install xapiand
```

### Build from Source

Requires CMake 3.5+, a C++17 compiler, and the [Xapian](https://xapian.org) library.

```sh
mkdir build && cd build
cmake ..
cmake --build . -j$(nproc)
```

### Run

```sh
mkdir -p my-database && cd my-database
xapiand -vvvv
```

By default, the HTTP REST API is available on port **8880**.

### Docker

Pre-built images are available on GitHub Container Registry:

```sh
docker pull ghcr.io/dubalu-development-team/xapiand:stable
docker run -p 8880:8880 ghcr.io/dubalu-development-team/xapiand:stable
```

To persist data across restarts, mount a host directory or a named volume:

```sh
# Using a host directory:
docker run -p 8880:8880 -v /path/to/data:/var/db/xapiand ghcr.io/dubalu-development-team/xapiand:stable

# Using a named volume:
docker run -p 8880:8880 -v xapiand-data:/var/db/xapiand ghcr.io/dubalu-development-team/xapiand:stable
```

To use a custom database path inside the container, set the `XAPIAND_DATABASE`
environment variable:

```sh
docker run -p 8880:8880 -e XAPIAND_DATABASE=/data -v /path/to/data:/data ghcr.io/dubalu-development-team/xapiand:stable
```

| Variable | Default | Description |
|---|---|---|
| `XAPIAND_DATABASE` | `/var/db/xapiand` | Database directory inside the container |
| `XAPIAND_CLUSTER` | Container hostname | Cluster name to join |

To build the image locally instead:

```sh
docker build -f contrib/docker/xapiand/Dockerfile -t xapiand:latest .
docker run -p 8880:8880 xapiand:latest
```


## Usage Examples

### Index a Document

```
PUT /twitter/user/Kronuz

{
  "name" : "German M. Bravo"
}
```

```
PUT /twitter/tweet/1

{
  "user": "Kronuz",
  "postDate": "2016-11-15T13:12:00",
  "message": "Trying out Xapiand, so far, so good... so what!"
}
```

### Retrieve & Search

```
GET /twitter/user/Kronuz

GET /twitter/tweet/1

GET /twitter/tweet/:search?q=user:Kronuz&pretty
```

### Store a File

```
STORE /twitter/images/Kronuz
Content-Type: image/png

@Kronuz.png
```

```
GET /twitter/images/Kronuz
Accept: image/png
```

The API supports standard HTTP methods (GET, POST, PUT, DELETE, HEAD, OPTIONS) plus MERGE and STORE. Both JSON and MessagePack content types are accepted.


## Command-Line Options

```
xapiand [options]
```

| Option | Description |
|---|---|
| `--cluster <name>` | Cluster name to join |
| `--name <node>` | Node name |
| `--solo` | Run solo indexer (no replication or discovery) |
| `-D, --database <path>` | Path to root of the node |
| `--http <port>` | HTTP REST API port (default: 8880) |
| `--xapian <port>` | Xapian binary protocol port |
| `--workers <threads>` | Number of worker servers |
| `--dbpool <size>` | Maximum number of databases in pool |
| `-d, --detach` | Run as daemon |
| `-v, --verbose` | Increase verbosity (can be repeated) |
| `--strict` | Force user to define type for each field |
| `--foreign` | Force foreign (shared) schemas for all indexes |
| `--uuid <mode>` | UUID mode: simple, compact, partition, encoded |
| `-h, --help` | Display usage information |


## Documentation

Full documentation is available at [https://kronuz.io/Xapiand](https://kronuz.io/Xapiand), covering:

- [Quick Start](https://kronuz.io/Xapiand/docs/quickstart/)
- [API Reference](https://kronuz.io/Xapiand/docs/reference-guide/api/)
- [Query DSL](https://kronuz.io/Xapiand/docs/reference-guide/query-dsl/)
- [Schema](https://kronuz.io/Xapiand/docs/reference-guide/schema/)
- [Storage](https://kronuz.io/Xapiand/docs/reference-guide/storage/)
- [Aggregations](https://kronuz.io/Xapiand/docs/reference-guide/aggregations/)
- [Scripting](https://kronuz.io/Xapiand/docs/reference-guide/scripting/)


## License

```
Copyright (C) 2015-2018 Dubalu LLC. All rights reserved.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to
deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
sell copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
IN THE SOFTWARE.
```
