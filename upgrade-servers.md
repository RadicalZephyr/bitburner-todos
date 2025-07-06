I want to turn the `src/buy-servers.ts` script into a service daemon.

Instead of upgrading all personal servers at once, the `ServerUpgrade`
service will upgrade one server at a time, hopefully with zero
downtime for any of the other services.

In order to make this work, the daemon will need to be able to
collaborate with the other services to clear any jobs running on the
server being targeted for upgrading.


# Buying Servers Initially

Buying servers initially is much simpler than upgrading. We have a
configurable `minimumServerRam` value, and when we have enough money
we buy a server until we can't buy any more.

If we wanted to get more sophisticated we could measure the rate at
which money is coming in `ns.getMoneySources().sinceInstall.hacking`
and estimate how expensive an initial server we can afford. We could
have a heuristic that balances how long it will take to earn the money
and how much faster we could earn it if we bought a lower memory server.


# Basic Outline for Upgrading a Server

## Choosing a server

Choosing a server to upgrade is pretty clear, sort by lowest RAM then
choose the server with the lowest allocated memory. Actually, lowest
allocated is just a useful proxy for what we really care about, which
server could be available the soonest without having to kill any of
the processes on it?

We could add a message to the `MemoryAllocator` to request which
server could be freed soonest.

Then we communicate with the `MemoryAllocator` and request that it
deallocate the RAM on that server.

Or, instead of this back-and-forth, we send the `MemoryAllocator` a
list of personal servers and it decides which one will be removed from
the memory pool.

If we made the Memory system more involved in launching tasks then we
could record the information about exactly what process is in each
allocation chunk.

This could be part of the refactor to make the Memory system more
encapsulated.

We don't necessarily need to send this information to the
`MemoryAllocator` when we get it. The allocator already stores which
allocations have chunks on which servers. When we want to find out
which server will terminate first we can request the process
information from each `MemoryClient` (but when do we force the user to
`await` the client so that it can receive these messages?).
