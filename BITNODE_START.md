# Handle Sudden Increases in RAM Better

Early in a run, getting access to new port cracking programs causes a
whole batch of new servers to become available from the Discovery
service, and each new port cracked increases the amount of RAM of most
servers in the new group.

Early in the run when we're heavily RAM constrained the tasks that we
have spawned are deliberately undersized to fit into available RAM.

Currently, when new RAM becomes available the TaskSelector simply
greedily selects the next task that hasn't been started. This means
that as we unlock larger amounts of RAM we start more tasks instead of
growing the undersized early tasks. This means that to get the
TaskSelector to focus on the correct tasks we need to restart it so
the greedy task selection algorithm can assign more RAM to the tasks
that are the farthest along.

This is a significant flaw in the batch hacking system and the
TaskSelector specifically. In order to fix it we need to introduce
some mechanism for allowing specific tasks to grow in size.


## Ideas

### Priorities

We want to be able to provide smooth growth through increasing amounts
of memory.

Minimizing downtime is important. Ideally, each task will be able to
respond to changing memory availability.

One situation that I want to be able to handle more gracefully is
upgrading personal servers. This is complex because in order to
upgrade the server there need to be zero scripts running on it. This
means we need some mechanism to be able to shift allocations around
to different hosts transparently.

### Memory Pooling?

We could incorporate memory pooling, but we kind of get a nice balance
right now emergently by the fact that we allocate harvesting batches
in chunks that contain all four phases (HWGW) so these chunks are much
larger than the chunk size of till and sow jobs (which have a chunk
size of one thread of the relevant scripts).

Since we prioritize allocation to harvest scripts first, we use the
majority of memory on harvesting, then we can fill in the gaps in
servers created by host boundaries, or fill up smaller servers that
can't fit even one harvest batch with sowing and till scripts.

### Till and Sow

With shrinkable allocations till and sow scripts can request the full
size allocation they want. If we create a better class encapsulation
of an allocation, then the memory manager can send new chunks of
memory to each allocation until it fulfills the max amount it asked
for.

`OwnedAllocations` should become a real class, internally they would
have a port (allocated dynamically from the `PortAllocator`) the
`MemoryAllocator` can send messages to. Then in the main loop of the
sow or till tasks they can simply loop through the chunks they have
and transparently end up spawning more tasks.

### Harvest

Growing a harvest task can work the same way up until the number of
chunks reaches maximum overlap. To keep growing it after that, you need
to start increasing the size of each batch which doesn't fit neatly
into the organization of the way we allocate memory.

It might be possible to reorganize some chunks to grow the chunk size
and shrink the number of chunks.

Perhaps the hacker could try to allocate larger batch size chunks?
That will almost certainly fail for early batches because we just
don't have the contiguous chunks to allocate for large batches.

More likely we need a reallocation mechanism where we can try
to... grow the allocation?

The flow for the harvest task could be something like:

- calculate the batch size for the command-line `--max-ram` (same as
  it does now)

- this is either a minimal size batch that potentially has incomplete
  overlap

- or a (mostly) full overlap batch that fits into the max-ram specified
