# Performance
Latency =  SUM(executionTimeAlongCriticalPath)

There are two ways to reduce latency:
1.	Decrease the number of tasks on the critical path.
2.	Shorten the time spent on each task:\
    a.	By reducing or eliminating network and disk usage \
    b.	By reducing execution time for each task

Let’s review the first point. As shown in the high-level design, the critical trading path includes the following:
## Gateway —> Order Manager —> Sequencer —> Matching Engine

In the high-level design, the components on the critical path run on individual servers connected over the network. The round trip network latency is about 500 microseconds. When there are multiple components all communicating over the network on the critical path, the total network latency adds up to single digit milliseconds. In addition, the sequencer is an event store that persists events to disk. Even assuming an efficient design that leverages the performance advantage of sequential writes, the latency of disk access still measures in tens of milliseconds.
Accounting for both network and disk access latency, the total end-to-end latency adds up to tens of milliseconds. While this number was respectable in the early days of the exchange, it is no longer sufficient as exchanges compete for ultra-low latency.
To stay ahead of the competition, exchanges over time evolve their design to reduce the end-to-end latency on the critical path to tens of microseconds, primarily by exploring options to reduce or eliminate network and disk access latency. A time-tested design eliminates the network hops by putting everything on the same server. When all components are on the same server, they can communicate via mmap as an event store.

![One Single Server](./assets/StockExchange_Mmap.svg)

There are a few interesting design decisions that are worth a closer look at.
An application loop is an interesting concept. It keeps polling for tasks to execute in a while loop and is the primary task execution mechanism. To meet the strict latency budget, only the most mission-critical tasks should be processed by the application loop. Its goal is to reduce the execution time for each component and to guarantee a highly predictable execution time (low 99th percentile latency). Each box in the diagram represents a component. A component is a process on the server. To maximize CPU efficiency, each application loop (think of it as the main processing loop) is single-threaded, and the thread is pinned to a fixed CPU core. 


## Application Loop in Order Manager
![Order Manager Application Loop](./assets/StockExchange_ApplicationLoop_OrderManager.svg)

In this diagram, the application loop for the order manager is pinned to CPU 1. The benefits of pinning the application loop to the CPU are substantial:
1.	No context switch. CPU 2 is fully allocated to the order manager’s application loop.
2.	No locks and therefore no lock contention, since there is only one thread that updates states.
Both of these contribute to a low 99th percentile latency.
The tradeoff of CPU pinning is that it makes coding more complicated. \
Engineers need to carefully analyze the time each task takes to keep it from occupying the application loop thread for too long, as it can potentially block subsequent tasks.

## mmap
mmap refers to a POSIX-compliant UNIX system call named mmap(2) that maps a file into the memory of a process.
Map(2) provides a mechanism for high-performance sharing of memory between processes. \
The performance advantage is compounded when the backing file is in /dev/shm. /dev/shm is a memory-backed file system. \
When mmap(2) is done over a file in /dev/shm, the access to the shared memory does not result in any disk access at all. \
Modern exchanges take advantage of this to eliminate as much disk access from the critical path as possible, mmap(2) is used in the server to implement a message bus over which the components on the critical path communicate. \
The communication pathway has no network or disk access, and sending a message on this mmap message bus takes sub-microsecond. \
By leveraging mmap to build an event store, coupled with the event sourcing design paradigm which we will discuss next, modern exchanges can build low-latency microservices inside a server. \
[wikipedia](https://en.wikipedia.org/wiki/Mmap)

## Event Sourcing
![Event Sourcing](./assets/StockExchange_EventSourcing.svg)

In the diagram, the external domain communicates with the trading domain using FIX.
-	The gateway transforms FIX to “FIX over Simple Binary Encoding" (SBE) for fast and compact encoding and sends each order as a NewOrderEvent via the Event Store Client in a pre-defined format.
-	The order manager (embedded in the matching engine) receives the NewOrderEvent from the event store, validates it, and adds it to its internal order states. Ihe order is then sent to the matching core.
-	If the order gets matched, an OrderFilledEvent is generated and sent to the event store.
- Other components such as market data publisher and reporter subscribe to the event store and process those events accordingly.

This design follows the hight level design closely, but there are some adjustments to make it work more efficiently in the event sourcing paradigm. \
The first difference is the order manager. The order manager becomes a resusable library that is embedded in different components. It makes sense for this design because the states of the orders are important for multiple components. Having a centralized order manager for other components to update or query the order states would hurt latency, especially if those components are not on the critical path. as is the case for the reporter in the diagram. Altough each component maintains the order states by itself, with event sourcing the states are guaranteed to be identical and replayable. \
Another key difference is that the sequencer is nowhere to be seen. With the event sourcing design. we have one single event store for all messages. Note that the event store entry contains a `Sequence` field. This field is injected by the sequencer. \
There is only one sequencer for each event store. It is bad practice to have multiple sequencers, as they will fight for the right to write to event store. In a busy system like an exchange, a lot of time would be wasted on lock contention. Therefore, the sequencer is a single writer sequences the events before sending them to the event store. \
Unlike the sequencer in the high-level design which also functions as a message store, the sequencer here only does one simple thing and is super fast. \
The sequencer pulls events from the ring buffer that is local to each component. For each event, it stamps a sequence ID on the event and sends it to the event store. We can have backup sequencers for high availability in case the primary sequencer goes down.

![Simple Sequencer](./assets/StockExchange_SimpleSequencer.svg)

We use Cirular buffering to achieve better perforemance on processing incoming requests. \
Circular avoids locking while writing data in buffer, this buffer going to be used by Gateway and Sequencer where in this design Reader of data is the Sequencer which pulls data out of buffer and process the request then writes them on Event Store(mmap).