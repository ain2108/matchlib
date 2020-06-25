## ScratchPad

### General Notes

#### Usage of SC_THEADs
The first thing that I noticed is that a SC_THREAD is being used as
opposed to a SC_CTHREAD. This is a bit confusing since I was under
impression that this is a synthesizable componenent.

#### Usage of Connections
The implementation relies on a the Connections library to
implement the request/respone channels of the scratchpad.
```cpp
Connections::In<cli_req_t<T, ADDR_WIDTH, N> > cli_req;
Connections::Out<cli_rsp_t<T, N> > cli_rsp;
```
`N` here seems to be the "Number of banks" == "Number of requests". 

#### Scratchpad structure

The connection allows to pass `N` objects of type `T`
through the LIC. The scratchpad can therefore process `N`
requests at the same time, by routing the `N` requests to their respective banks.

### Questions
1. Why the run method of Scratchpad is a SC_THREAD?
2. A bit confused with all the passes by value? Eg. crossbar call.
3. 

## Crossbar

### General Notes

#### Implementation

### Questions
1. Performance of the crossbar is a bit confusing to me. Is it fully combinational?

## Conversation with Davide
### Mental model
Before getting futher, I would like to arrive at a better definition of the problem and its constraints.
So I have the following description:
1. The accelerator has the classic load-compute-store phases.
2. Compute phase reads from input PLM and writes into a different output PLM.
3. Compute phase is trivially parallelizable (no data access conflicts) by adding more computational resources (more SC_CTHREADS in systemc).
Problem: As the compute processes are replicated so must be the PLM blocks that these processes are reading from and writing to.
Solution: We notice with accelerators that have a significant compute phase, the PLM blocks spends most of their cycles idle. Instead of replicating PLMs as the compute processes are replicated, we are going to minimize the idle cycles of a single PLM block by serializing the PLM accesses of the compute processes.

When are we successful?
1. An abritrated scratchpad is integrated with a real world accelerator.
2. Limiting PLM duplication to x1, the arbiter integration allows for performance improvement due to parallelization of the compute phase that cannot be achieved without the use of arbiter. All memories are assumed to be dual-port. Pipelining is not allowed (more on this later).
3. Integration is flexible, requiring no code changes as the parallelization of compute phase is changed.

### Choosing a real-world accelerator
I would like to be careful while choosing a real world accelerator so integration has even a theoretical chance to be worthwhile.
This really burns down to few constraints on the accelerator:
1. Compute phase is longer than 4 cycles (this is just from my research during the Spring semester)
2. Next access address is known ahead of time. This allows us to negate the arbiter latencies by not waiting for request completion.
3. Datapath should be complex enough to preven effective and automatic pipelining. If pipelining can be done easily, the PLM can be kept at 100% utilization as a new value is requested at each clock cycle. This of course significantly weakens the argument for a PLM arbiter. Arbiter only wins if the PLM is kept mostly idle during normal access patterns.


## Links
1. [Round-Robin Arbiter](https://rtlery.com/articles/how-design-round-robin-arbiter)
2. [MS Paper of Aung Toe](https://scholarworks.rit.edu/cgi/viewcontent.cgi?article=10982&context=theses)