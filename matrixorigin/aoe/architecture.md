- Feature Name: Analytic Optimized Engine
- Status: In Progress
- Start Date: 2021-05-10
- Authors: [Xu Peng](https://github.com/XuPeng-SH)
- PR:
- Issue:

## Summary

**AOE**(Analytic Optimized Engine) is designed for analytical query workloads, which can be used as the underlying storage engine of database management system (DBMS) for online analytical processing of queries (OLAP).
- Cloud native
- Efficient processing small and big inserts
- State-of-art data layout for vectorized execution engines
- Fine-grained, efficient and convenient memory management
- Conveniently support new index types and provide dedicated index cache management
- Serious data management for concurent queries and mutations
- Avoid external dependencies

## Terminology
### Layout
- **Block**: Piece of a segment which is composed of columns
- **Segment**: Piece of a table which is composed of blocks
- **Table**: Piece of a database which is composed of segments
- **Database**: A combination of tables, which shares the same log space

### Container
- **Vector**: Data fragment of a column in memory
- **Batch**: A combination of vectors, and the number of rows in each vector is aligned

### Misc
- **Log space**: A raft group can be considered as a log space

## Guilde-level design

## Reference-level design

### Detailed design

### Pros & Cons

### Open questions
