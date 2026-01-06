# Engineering Brief: In Search of an Understandable Consensus Algorithm
> **Source:** [Raft paper](https://raft.github.io/raft.pdf)  
> **Topic:** Distibuted Consensus Algorithm

## 1. Abstract Analysis
- **Core Problem:** The problem they has was that the previous consensus algorithm - Paxos was too difficult to understand and has a high complexity of implementation and somehow people were implementing something else instead of following the Paxos protocol. This led to "Shadow Implementations" where engineers built unverified protocols that differed from the original paper, creating unreliable systems.
- **Architectural Solution:** Raft decomposes consensus into three independent sub-problems: **Leader Election**, **Log Replication**, and **Safety**. This modularity makes the state space easier to reason about without sacrificing performance.
- **Constraints:** The primary constraint was **Understandability**. The design had to be intuitive enough for a system architect to implement a provably correct version from the paper alone.

## 2. Technical Primitives
*Definitions of the fundamental concepts required for implementation:*
- **Replicated State Machine:** A design where multiple nodes compute identical copies of the same state by executing the same sequence of commands from a replicated log. If the state machines are deterministic, the final state across all nodes remains consistent. This is figure 1 of research paper. state machine can be considered as database where we do operations based on logs. and this machine is replicated to multiple nodes. so this is called relicated state machines.
- **Shard:** A logical slice of the global dataset (e.g., Keys A-L). Sharding allows a database to distribute data across multiple nodes rather than requiring one node to hold the entire dataset. This is a conceptual term, this represent the the data which needs to be stored (unique data). like i need to store key-value pairs from A-Z then A-L and L-Z can 2 different shards.
- **Relication:** As the paper describes, one Raft group which has some configurable node (for example 5), now each operations is replicated to each node according to logs. Now if we observe, we can total 5 replication of each state machine (data). So, if i want to store 10 GB of unique data then i have to buy 50 GB. So, there he 40 GB overhead. now it shows drawback of Raft that we have alot of overhead.
But many time we need this overlead because we can't lose the data. Data is so precious that if it get deleted or corrrupted it will cost millions.
- **MultiRaft** - Above paper only describe single raft group. but in real world, we will be using multi-raft. Difference between multi-raft and raft group - for exampe take 50 severs and each with 10 GB storage. So total storage 50*10 = 500 GB. Now if we use single raft group and connect all 50 server as one Raft group then it will be 50 times replicationa and only 10 GB unique data can be store and 490 GB will be wasted in replication. now we can also think of mutlit-raft in 2 different ways. connecting 5 different severs and taking whole 10GB as shard and 100MB as shard.
which to prefer and why?
Analysis - Mathematically both will be same but Architecually they aren't.
### Shard Size Analysis: The "Goldilocks" Problem
| Shard Size | Number of Shards | Recovery Speed | Metadata Overhead | System Stability |
| :--- | :--- | :--- | :--- | :--- |
| **1 MB** | 100,000 | Instant | Extremely High | 🔴 **Risky:** CPU/Network saturated by heartbeats. |
| **100 MB** | 1,000 | Very Fast | Low/Moderate | 🟢 **Ideal:** Industry standard (TiDB/CockroachDB). |
| **1 GB** | 100 | Moderate | Negligible | 🟡 **Good:** Suitable for simpler NoSQL systems. |
| **10 GB** | 10 | Slow | Minimal | 🟠 **Poor:** Leads to fragmentation and "Hotspots." |

### Why Multi-Raft (Small Shards) Wins:
1. **MTTR (Mean Time To Recovery):** Small shards allow 100 servers to participate in re-replicating data from a failed node in parallel, rather than one server choking on a 10GB file.
2. **Hotspot Mitigation:** Spreads traffic across the entire cluster.
3. **Fluid Scaling:** Adding a new server is easier when you can move 100MB "grains of sand" to balance the load perfectly.

## 3. System Design & State Transitions
### Logical Components
- **[Module Name]:** [Function and state responsibility]
- **[Module Name]:** [Function and state responsibility]

### Architectural Diagram
[Insert technical diagram or Mermaid.js chart]

## 4. Implementation Log
### Design Decisions
- **Concurrency Model:** [e.g., Actor-based, Multi-threaded with Shared State]
- **Network Layer:** [e.g., TCP with custom binary serialization / gRPC]

### Critical Path Analysis
- **Hurdle:** [Technical description of the most complex logic, e.g., Log Matching Property]
- **Resolution:** [The algorithmic approach taken to resolve it]

## 5. Benchmarks & Validation
- **Correctness:** [Description of test cases for failure recovery]
- **Performance:** [Latency/Throughput results]

---
## 6. References
- [Technical Documentation]