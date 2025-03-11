# Hierarchy of Data Storage in Ceph: How Data is Written to a Ceph Cluster

When a client writes data to a Ceph cluster, the data follows a hierarchical flow involving Monitor (MON) nodes, Object Storage Daemons (OSDs), CRUSH algorithm, Placement Groups (PGs), and Storage Pools. Below is a step-by-step explanation of how the data flows through these components.

## Step-by-Step Data Write Flow in Ceph

1. **Client Initiates Write Operation**  
    A Ceph client (such as RADOS, CephFS, or RGW) wants to write an object. It does not write directly to a specific OSD; instead, it first determines where the data should go.

2. **Contacting Monitor (MON) Nodes**  
    The client first contacts a Monitor (MON) node to retrieve the latest cluster map, which includes:
    - The OSD map (which OSDs are up and in the cluster).
    - The PG map (which Placement Groups exist).
    - The CRUSH map (how data is distributed).  
    🔹 MON nodes do NOT store data; they only provide metadata and ensure cluster consistency.

3. **Determining Placement Group (PG) Using CRUSH Algorithm**  
    The client determines where the data should be stored using the CRUSH (Controlled Replication Under Scalable Hashing) algorithm.  
    The CRUSH algorithm maps the object to a Placement Group (PG) using the following formula:

    ```
    PG = hash(object_name) % total_number_of_PGs
    ```

    Example:  
    If the object name is "data1", Ceph hashes it and assigns it to PG 7. The client does this calculation without needing to query the MON (as long as it has the latest cluster map).

4. **Placement Group (PG) to Object Storage Daemon (OSD) Mapping**  
    Each Placement Group (PG) is assigned to one or more OSDs. The primary OSD for that PG is responsible for handling writes and replicating data to secondary OSDs.  
    Example:  
    If PG 7 is mapped to OSDs 3, 5, and 9, then:
    - OSD 3 is the primary OSD.
    - OSDs 5 and 9 are replica OSDs.

5. **Writing Data to Primary OSD**  
    The client directly writes data to the primary OSD of the selected PG. The primary OSD acknowledges the write and then replicates the data to the secondary OSDs.

6. **Replication to Secondary OSDs (if replication factor > 1)**  
    The primary OSD sends copies of the data to secondary OSDs as per the replication factor.  
    Example:  
    If the replication factor = 3, the primary OSD writes the object and sends copies to two secondary OSDs. The primary OSD waits for acknowledgment from secondary OSDs before confirming a successful write.

7. **Writing Data to Storage Pools**  
    OSDs store the data in pools, which are logical groups of PGs. Each pool has:
    - Replication factor (e.g., 3 copies).
    - CRUSH rule (which dictates data placement across OSDs).  
    Data is physically written to disk in OSD storage backends (e.g., BlueStore).

8. **Acknowledgment to Client**  
    Once the primary OSD receives acknowledgment from all secondary OSDs, it sends an acknowledgment to the client. Now, the write is considered successful.

## Full Hierarchy of Data Flow in Ceph

1️⃣ **Client**  
    Requests a write operation.

2️⃣ **Monitor (MON)**  
    Provides cluster metadata (OSD map, PG map, CRUSH map).

3️⃣ **CRUSH Algorithm**  
    Determines the PG for the object.

4️⃣ **Placement Group (PG)**  
    Maps to primary and replica OSDs.

5️⃣ **Primary OSD**  
    Accepts the write, replicates to secondary OSDs.

6️⃣ **Secondary OSDs**  
    Store additional copies (if replication >1).

7️⃣ **Pools**  
    Logical storage of PGs containing data.

8️⃣ **Final Acknowledgment**  
    Once all OSDs confirm, the client receives a success message.

## Diagram Representation

```plaintext
Client  --->  Monitor (MON)  --->  CRUSH Algorithm  --->  Placement Group (PG)  --->  Primary OSD
                                                                                    |
                                                              Replication to Secondary OSDs
                                                                                    |
                                                                          Storage Pools
                                                                                    |
                                                                     Data written to disk
```

## Key Takeaways

✅ Ceph does not rely on a centralized metadata server; the client calculates where to store data using CRUSH.  
✅ The Monitor nodes do NOT handle data directly; they provide cluster metadata.  
✅ Placement Groups (PGs) act as an intermediary between pools and OSDs to ensure data is evenly distributed.  
✅ Primary OSD handles writes and replicates data to secondary OSDs before confirming success.  
✅ Ceph automatically balances and recovers data if an OSD fails.