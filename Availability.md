# Availability
For high availability, our design aims for (`99.99%`). \
This means the exchange can only have `8.64` seconds of downtime per day. \
It requires almost immediate recovery if a service goes down. \
To achieve high availability; consider the following:
- First, identify single-point-of-failures in the exchange architecture. For example, the failure of the matching engine could be a disaster for the exchange. Iherefore, we set up redundant instances alongside the primary' instance.
- Second, detection of failure and the decision to failover to the backup instance should be fast.
For stateless services such as the client gateway, they could easily be horizontally scaled by adding more servers.

![Hot Warm Matching Engine](./assets/StockExchange_HotWarmMatchingEngine.svg)

For stateful components, such as the order manager and matching engine, we need to be able to copy state data across replicas. \
The hot matching engine works as the primary’ instance, and the warm engine receives and processes the exact same events but does not send any event out onto the event store. When the primary goes down, the warm instance can immediately take over as the primary' and send out events. When the warm secondary' instance goes down, upon restart, it can always recover all the states from the event store. Event sourcing is a great fit for the exchange architecture. The inherent determinism makes state recovery easy and accurate.

## Fault Tolerance
The hot-warm design above is relatively simple. It works reasonably well, but what happens if the warm instances go down as well? \
This is a low probability but catastrophic event, so we should prepare for it. \
This is a problem large tech companies face. They tackle it by replicating core data to data centers in multiple cities. \
It mitigates the risk of a natural disaster such as an earthquake or a large-scale power outage. \
To make the system fault-tolerant, we have to answer many questions:
1.	If the primary instance goes down, how and when do we decide to failover to the backup instance?
2.	How do we choose the leader among backup instances?
3.	What is the recovery time needed (RTO - Recovery Time Objective)?
4.	What functionalities need to be recovered (RPO - Recovery Point Objective)? Can our system operate under degraded conditions?

### What 'down' means?
1. The system might send out false alarms, which cause unnecessary failovers.
2. Bugs in the code might cause the primary instance to go down. The same bug could bring down the backup instance after the failover. When all backup instances are knocked out by The bug, the system is no longer available.

When we first release a new system, we might need to perform failovers manually. Only when we gather enough signals and operational experience and gain more confidence in the system do we automate the failure detection process. \
Chaos engineering is a good practice to surface edge cases and gain operational experience faster.
Once the decision to failover is correctly made, how do we decide which server takes over? \
Fortunately, this is a well-understood problem. There are many battle-tested leader-election algorithms. We use Raft as an example. 
[More Info -  Chatgpt](./BattleTestedLeaderElectionAlgorithms.md)

![Raft](./assets/StockExchange_EventReplicationWithRaft.svg)

Raft cluster with 5 servers with their own event stores. The current leader sends data to all the other instances (followers). \
The minimum number of votes required to perform an operation in Raft is `n / 2 + 1`, where `n` is the number of members in the cluster. In this example, the minimum is `5 / 2 + 1 = 3`
the followers receiving new events from the leader over RPC. The events are saved to the follower’s own mmap event store.

- The leader sends heartbeat messages (`AppendEnties` with no content) to its followers.
- If a follower has not received heartbeat messages for a period of time, it triggers an election timeout that initiates a new election.
- The first follower that reaches election timeout becomes a candidate, and it asks the rest of the followers to vote (RequestVote).
- If the first follower receives a majority of votes, it becomes the new leader.
- If the first follower has a lower term value than the new node, it cannot be the leader.
- If multiple followers become candidates at the same time, it is called a “split vote". In this case, the election times out. and a new election is initiated.

![Raft Terms](./assets/StockExchange_RaftTerms.svg)

Time is divided into arbitrary intervals in Raft to represent normal operation and election.

Recovery Time Objective (RTO) refers to the amount of time an application can be down without causing significant damage to the business. \
For a stock exchange, we need to achieve a second-level RTO, which definitely requires automatic failover of services. \
To do this, we categorize services based on priority and define a degradation strategy to maintain a minimum service level. \
Recovery Point Objective (RPO) refers to the amount of data that can be lost before significant harm is done to the business. \
In practice, this means backing up data frequently. For a stock exchange, data loss is not acceptable, so RPO is near zero. \
With Raft, we have many copies of the data, it guarantees that state consensus is achieved among cluster nodes. \
If the current leader crashes, the new leader should be able to function immediately.