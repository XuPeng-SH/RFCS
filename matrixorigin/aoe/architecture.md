- Feature Name: Analytic Optimized Engine
- Status: In Progress
- Start Date: 2021-05-10
- Authors: [Xu Peng](https://github.com/XuPeng-SH)
- PR:
- Issue:

## Summary

**AOE** (Analytic Optimized Engine) is designed for analytical query workloads, which can be used as the underlying storage engine of database management system (DBMS) for online analytical processing of queries (OLAP).
- Cloud native
- Efficient processing small and big inserts
- State-of-art data layout for vectorized execution engines
- Fine-grained, efficient and convenient memory management
- Conveniently support new index types and provide dedicated index cache management
- Serious data management for concurent queries and mutations
- Avoid external dependencies

## Guilde-level design

### Terms
#### Layout
- **Block**: Piece of a segment which is composed of columns
- **Segment**: Piece of a table which is composed of blocks
- **Table**: Piece of a database which is composed of segments
- **Database**: A combination of tables, which shares the same log space

#### Container
- **Vector**: Data fragment of a column in memory
- **Batch**: A combination of vectors, and the number of rows in each vector is aligned

#### Misc
- **Log space**: A raft group can be considered as a log space

### Write path
#### DDL
1. Equeue the incoming request
2. Schedule the request to catalog
3. Catalog validate the request and start a transaction Txn
4. Prepare and save mutations to the Txn
5. Commit the Txn to LogStore

#### DML
1. Equeue the incoming request
2. Validate the request
3. Fetch the specified table mutation handle to process mutation request
   1) Wrap a WAL entry and flush to the LogStore
   2) Apply the mutation and make it public

### Read path
#### DDL
1. Equeue the incoming request
2. Schedule the request to catalog
3. Catalog validates the request
4. Catalog executes read options

#### DQL
1. Equeue the incoming request
2. Get a table data snapshot base on timestamp. If timestamp is missing, always get the latest snapshot
3. Initialize table readers wrapping the snapshot for parallel loading

### Data storage

### Components
#### WAL
#### Buffer manager
#### Catalog: In-memory metadata manager
#### LogStore: A embedded log-structured data store
#### Tables
#### Reader

## Reference-level design

### Detailed design

### Pros & Cons

### Open questions
