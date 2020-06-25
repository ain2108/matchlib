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

## Links
1. [Round-Robin Arbiter](https://rtlery.com/articles/how-design-round-robin-arbiter)
2. [MS Paper of Aung Toe](https://scholarworks.rit.edu/cgi/viewcontent.cgi?article=10982&context=theses)