# Thin Provisioning

- Allows for an economical allocation and utilization of storage space by moving arbitrary data blocks to contiguous locations, which results in empty block elimination. 
- Can create a thin pool of storage space and assign volumes much larger storage space than the physical capacity of the pool. 
- Workloads begin consuming the actual allocated space for data writing. 
- When a preset custom threshold (80%, for instance) on the actual consumption of the physical storage in the pool is reached, expand the pool dynamically by adding more physical storage to it. 
- The volumes will automatically start exploiting the new space right away.
- helps prevent spending more money upfront.

