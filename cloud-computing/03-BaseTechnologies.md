# 03 BASE TECHNOLOGIES

## Trends in Processor Architecture
* Continuous decrease in transistor size
* Tick-tock model by Intel: Tick (shrink processor size), Tock (frame new architecture) every 12 to 18 months

## Accelerator Programming
* Objectives:
    * Increase computational speed
    * Reduce energy consumption
* 3 Types:
    1. GPGPUs (General Purpose Graphics Processing Units)
    2. Xeon Phi 
    3. FPGA (Field Programmable Gate Array)

## Memory
### Single physical address space
* **Volatile**: Requires power to maintain the currently stored information.
* **Non-volatile** (NVM): Stored information can be retrieved after turning storage device off and back on.

### Shared Memory Systems
* [Shared memory](https://en.wikipedia.org/wiki/Shared_memory) can be accessed by multiple programs simultaneously with the intent to share data. 
* NUMA: Multiple CPUs with multiple cores and single physical address spaces.

### Distributed Memory System
* Each processor has its own memory. 

### Memory Hierarchy
```
Processor Registers      | Size/Capacity
Processor Cache          |
RAM                      |   ^ 
Flash Memory             V   | Access speed
Harddrives (Disk)            | Cost
Tape Backup                  |
```

## Storage

### RAID
Redundant Array of Independent Disks
* RAID 0: Striping
* RAID 1: Mirroring
* RAID 5: Striping and Parity Blocks

### Flash Memory
* Non-volatile

### Storage Provisioning
* Direct Attached Storage (to the individual computer)
* Storage Area Network SAN 
* Network Attached Storage NAS

#### SAN
* Storage Area Network
* A network provides access to block level data storage.
* On block level data storage, raw volumes of storage are created which can then be controlled as individual harddrives.
* This is typically a network separated from LAN.

#### NAS
* Network Attached Storage
* Storage is provided over a file server that is attached to the network


## Network
* Transport time for a message:
    * Physical delay
    * Protocol delay
    * Line waiting time
    * Transmission time

### Link Technologies
* Ethernet with VLANs: 10 Gbit/s
    * VLAN (Virtual Local Area Network): Single LAN is partitioned into multiple VLANs with own single broadcast domains.
* Infiniband 300 Gbit/s
* WLAN 802.11 ac: 1733 Mbit/s at 5 GHz
* DSL (VDSL: Very high bit rate digital subscriber): Upstream 16 Mbit/s, downstream 52 Mbit/s
* GSM (Global system for mobile communications) 4G: Uplink 75 Mbit/s, downlink 300 Mbit/s

## Fault tolerance
* **Fault**:
    * Defect of a system that may cause an error
* **Error**:
    * Illegal state within the system
* **Failure**:
    * Occurs after an error reaches the service interface of a system and results in inconsistency with specified system behavior.
* **Fault tolerance**:
    * A **fault tolerant** system never enters a state of failure, i.e. errors never reach the service interface.
    * Implementation steps:
        * Error detection
        * Error analysis
        * Recovery
* **Reliability**:
    * Probability that a system is free of failures up to a certain point in time.
* **Availability**:
    * Percentage of planned operation time in which the system is available.

## Security
The absence of non-permitted (maybe illegal) use of a system.

### Techniques of security management
1. Encryption
2. Digests (= 9. proof that message has not been changed)
3. Intrusion detection
4. Event recording for later auditing
5. Firewalls
6. Physical protection
7. Authentification / authentication (Proof of identity)
8. Authorization (Selective access control)
9. Digital signature (guarantee of intactness of a document)
10. Certificate (trustworthy agency confirms stuff)