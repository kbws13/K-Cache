# K-Cache

A high-performance, distributed cache system inspired by Google's BigTable caching architecture.

## Overview

K-Cache is a Go-based caching system that implements a distributed cache with HTTP-based peer communication. It features LRU eviction, consistent hashing for load distribution, and single-flight mechanism to prevent duplicate requests.

## Project Structure

```
.
├── http_server/          # HTTP server implementation for peer communication
│   └── http.go
├── kcache/               # Core cache implementation
│   ├── consistenthash/   # Consistent hashing algorithm
│   ├── lru/              # LRU cache implementation
│   ├── byteview.go       # Immutable byte view
│   ├── cache.go          # Cache structure
│   ├── kcache.go         # Main cache group logic
│   ├── peers.go          # Peer picker interfaces
│   └── ...
├── protobuf/             # Protocol buffer definitions
│   └── kcachepb.proto
├── singleflight/         # Duplicate request suppression
├── main.go               # Entry point
└── run.sh                # Run script
```


## Features

- **LRU Cache**: Implements Least Recently Used eviction policy
- **Distributed Caching**: HTTP-based peer communication for distributed caching
- **Consistent Hashing**: Even distribution of keys across cache nodes
- **Single Flight**: Prevents duplicate requests for the same key
- **Thread Safe**: Concurrent access protection with mutexes

## Usage

1. **Installation**
   ```bash
   go mod tidy
   ```


2. **Running the cache server**
   ```bash
   ./run.sh
   ```

   or
   ```bash
   go run main.go
   ```


3. **Creating a cache group**
   ```go
   import "kcache/kcache"
   
   var db = map[string]string{
       "Tom":  "630",
       "Jack": "589",
       "Sam":  "567",
   }
   
   getter := kcache.GetterFunc(func(key string) ([]byte, error) {
       if v, ok := db[key]; ok {
           return []byte(v), nil
       }
       return nil, fmt.Errorf("%s not exist", key)
   })
   
   group := kcache.NewGroup("scores", 2<<10, getter)
   ```


4. **Getting values**
   ```go
   if view, err := group.Get("Tom"); err == nil {
       fmt.Println(view.String())
   }
   ```


## Advantages

- **High Performance**: In-memory caching with O(1) access time
- **Scalable**: Distributed architecture with consistent hashing
- **Fault Tolerant**: Handles node failures gracefully
- **Easy Integration**: Simple API for integration with existing systems
- **Memory Efficient**: LRU eviction prevents memory overflow
- **Duplicate Request Prevention**: Single-flight mechanism reduces backend load

## HTTP Interface

The cache exposes an HTTP interface for peer communication:
```
GET /_kcache/<groupname>/<key>
```