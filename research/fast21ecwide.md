# fast21|ecwide

- Erasure coding parameters:medium range
- n-k:3/4
- repair penalty
- Near-optimal redundancy
- significantly reduce expensive hardware footprints
- parity locality
- topology locality
- combined locality
- Key:repair
  - single chunk
  - Full-node

- Encoding:using multiple nodes
- inner-rack parity update
- storage system that organize data in fixed-size chunks
- RS codes->redundancy is minimum
- Stripe->n chunks in n different nodes
- parity chunks:linear combination of data chunks based on the arithmetic of the Galois Field
- cauchy rs codes:encoding coeefficients come from Cauchy matrix
- Repair:bandwidth and io costs are amplified k times
- Hot system
  - frequent access->tail latency

- encoding
- update
- parity locality:reduces the repair bandwidth but incurs high redundancy.
- at the expense of
- topology locally:keep redundancy,achieves the minimum redundancy, but incurs high cross-rack repair bandwidth.
- adding linear dependency does not improve fault tolerance

- real global parity:to tolerate the max number of data chunks
- for a given level of redundancy, adding linear dependency does not improve fault tolerance

- for global parity:too low