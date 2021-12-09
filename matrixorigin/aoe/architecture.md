- Feature Name: Analytic Optimized Engine
- Status: In Progress
- Start Date: 2021-05-10
- Authors: [Xu Peng](https://github.com/XuPeng-SH)
- PR:
- Issue:

# Summary

**AOE** (Analytic Optimized Engine) is designed for analytical query workloads, which can be used as the underlying storage engine of database management system (DBMS) for online analytical processing of queries (OLAP).
- Cloud native
- Efficient processing small and big inserts
- State-of-art data layout for vectorized execution engines
- Fine-grained, efficient and convenient memory management
- Conveniently support new index types and provide dedicated index cache management
- Serious data management for concurent queries and mutations
- Avoid external dependencies

# Guilde-level design

## Terms
### Layout
- **Block**: Piece of a segment which is the minimum part of table data. The maximum number of rows of a block is fixed.
- **Segment**: Piece of a table which is composed of blocks. The maximum number of blocks of a segment is fixed.
- **Table**: Piece of a database which is composed of segments
- **Database**: A combination of tables, which shares the same log space

### State
- **Transient Block**: Block where the number of rows does not reach the upper limit and the blocks queued to be sorted and flushed
- **Persisted Block**: Sorted block
- **Unclosed Segment**: Segment that not merge sorted
- **Closed Segment**: Segment that merge sorted

### Container
- **Vector**: Data fragment of a column in memory
- **Batch**: A combination of vectors, and the number of rows in each vector is aligned

### Misc
- **Log space**: A raft group can be considered as a log space

## Write
### DDL
1. Equeue the incoming request
2. Schedule the request to catalog
3. Catalog validate the request and start a transaction Txn
4. Prepare and save mutations to the Txn
5. Commit the Txn to LogStore

### DML
1. Equeue the incoming request
2. Validate the request
3. Fetch the specified table mutation handle to process mutation request
   1) Wrap a WAL entry and flush to the LogStore
   2) Apply the mutation and make it public

## Read
### DDL
1. Equeue the incoming request
2. Schedule the request to catalog
3. Catalog validates the request
4. Catalog executes read options

### DQL
1. Equeue the incoming request
2. Get a table data snapshot base on timestamp. If timestamp is missing, always get the latest snapshot
3. Initialize table readers wrapping the snapshot for parallel loading

## Data storage
### Table
**AOE** stores data represented as tables. Each table is bound to a schema consisting of numbers of column definitions. A table data is organized as a log-structured merge-tree (LSM tree).

Currently, **AOE** is a three-level LSM tree, called L0, L1 and L2. L0 is small and can be entirely resident in memory, whereas L1 and L2 are both definitely resident on disk. In **AOE**, L0 consists of transient blocks and L1 consists of sorted blocks. The incoming new data is always inserted into the latest transient block. If the insertion causes the block to exceed the maximum row count of a block, the block will be sorted by primary key and flushed into L1 as sorted block. If the number of sorted blocks exceed the maximum number of a segment, the segment will be sorted by primary key using merge sort.

L1 and L2 are organized into sorted runs of data. Each run contains data sorted by the primary key, which can be represented on disk as a single file. There will be overlapping primary key ranges between sort runs. The difference of L1 and L2 is that a run in L1 is a **block** while a run in L2 is a **segment**.

As described above, transient blocks can be entirely resident in memory, but not necessarily so. Because there will be many tables, each table has transient blocks. If they are always resident in memory, it will cause a huge waste. In **AOE**, transient blocks from all tables share a dedicated fixed-size LRU cache. A evicted transient block will be unloaded from memory and flushed as a transient block file. In practice, the transient blocks are constantly flowing to the L1 and the number of transient blocks per table at a certain time is very small, those active transient blocks will likly reside in memory even with a small-sized cache.

### Indexes
There's no table-level index in **AOE**, only segment and block-level indexes are available. **AOE** supports dynamic index creation and deletion, index creation is an asynchronous process. Indexes will only be created on blocks and segments at L1 and L2, that is, transient blocks don't have any index.

In **AOE**, there is a dedicated fixed-size LRU cache for all indexes. Compared with the original data, the index occupies a limited space, but the acceleration of the query is very obvious, and the index will be called very frequently. A dedicated cache can avoid a memory copy when being called.

Currently, **AOE** uses two index types:
- **Zonemap**: It is automatically created for all columns. Persisted.
- **BSI**: It should be explicitly defined for a column. Persisted.

It is very easy to add a new index to **AOE**.

### Compression
**AOE** is a column-oriented data store, very friendly to data compression. It supports per-column compression codecs and now only **LZ4** is used. You can easily obtain the meta information of compressed blocks. In **AOE**, the compression unit is a column of a block.

### Layout
#### Block
   ![image](https://user-images.githubusercontent.com/39627130/145402878-72f9aa0a-65f5-494a-96ff-c075065c1f01.png)

#### Segment
   ![image](https://user-images.githubusercontent.com/39627130/145402537-6500bcf4-5897-4dfa-b3fc-196d0c5835df.png)


## Buffer manager

## WAL

## Catalog: In-memory metadata manager

## LogStore: A embedded log-structured data store

# Pros & Cons

# Feature works
