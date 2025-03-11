# RADOS (Reliable Autonomic Distributed Object Store)

RADOS is the core storage system of Ceph. It is the foundation upon which all Ceph services (e.g., RBD, RGW, CephFS) are built. RADOS is responsible for managing data storage, replication, and recovery in a distributed and fault-tolerant manner.


## What is RADOS?

RADOS is a distributed object storage system that provides the underlying storage layer for Ceph. It is designed to be scalable, fault-tolerant, and self-healing. RADOS manages the storage of objects (data) across a cluster of nodes, ensuring data durability and availability.



## Key Features of RADOS

### Distributed and Scalable

- RADOS distributes data across multiple nodes (servers) in the cluster.
- It can scale from a few nodes to thousands of nodes, handling petabytes or even exabytes of data.

### Fault-Tolerant

- Data is replicated or erasure-coded across multiple nodes, ensuring durability even if some nodes fail.
- RADOS automatically detects and recovers from failures.

### Self-Healing

- If a node or disk fails, RADOS automatically re-replicates data to maintain the desired level of redundancy.

### Autonomic

- RADOS manages itself with minimal human intervention, handling tasks like data rebalancing and failure recovery.

### Object-Based Storage

- RADOS stores data as objects, which are binary blobs with metadata.
- Each object is identified by a unique ID and stored in a pool.

## Components of RADOS

### OSD (Object Storage Daemon)

OSDs are the workhorses of RADOS. Each OSD manages data stored on a physical or logical disk. OSDs are responsible for:

- Storing objects.
- Handling read/write operations.
- Replicating data to other OSDs.

### Monitor (MON)

MONs maintain the cluster map, which includes information about OSDs, placement groups (PGs), and the CRUSH map. MONs ensure consistency and coordination across the cluster.

### CRUSH Algorithm

CRUSH (Controlled Replication Under Scalable Hashing) is the algorithm used by RADOS to determine how data is distributed across OSDs. It ensures even data distribution and efficient data placement.

### Placement Groups (PGs)

PGs are logical groupings of objects that are mapped to OSDs. RADOS uses PGs to distribute data evenly across the cluster.

## How RADOS Works

### Data Storage

- Data is stored as objects in RADOS pools.
- Each object is replicated or erasure-coded across multiple OSDs.

### Data Distribution

- The CRUSH algorithm determines which OSDs should store each object.
- Objects are grouped into PGs, and PGs are mapped to OSDs.

### Data Access

- Clients interact with RADOS using the librados library.
- RADOS handles read/write operations, ensuring data consistency and durability.

### Failure Recovery

- If an OSD fails, RADOS automatically re-replicates the data to other OSDs.
- The cluster remains operational even during failures.

## RADOS and Ceph Services

RADOS is the foundation for all Ceph services:

### RBD (RADOS Block Device)

- Provides block storage by mapping RADOS objects to block devices.

### RGW (RADOS Gateway)

- Provides object storage with an S3-compatible API.

### CephFS (Ceph File System)

- Provides a POSIX-compliant distributed file system built on RADOS.

## Example Workflow

1. A client writes data to a Ceph cluster.
2. RADOS stores the data as objects in a pool.
3. The CRUSH algorithm determines which OSDs should store the objects.
4. RADOS replicates the objects across multiple OSDs for durability.
5. If an OSD fails, RADOS re-replicates the data to other OSDs.

## Why is RADOS Important?

RADOS provides the scalability, fault tolerance, and self-healing capabilities that make Ceph a powerful distributed storage system. It abstracts the complexity of distributed storage, allowing Ceph to provide unified storage (block, object, and file) on a single platform.
