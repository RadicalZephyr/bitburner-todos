I just updated the structure of the host discovery service
(`src/services/discover.ts`), creating a pull API where other daemons
can send a request for the current list of workers or targets
available. This API also allows clients requesting these lists to
register a subscription to be notified of new hosts of that type. I
updated the `MemoryManager` service (`src/services/memory.tsx`), the
batch hacking `TargetSelector` service (`src/batch/manager.ts`) and
the batch hacking `Monitor` service (`src/batch/monitor.tsx`) to use
this new API and also to subscribe to newly discovered hosts of the
types that they care about.

I chose to have the `MemoryManager`, `TargetSelector` and `Monitor` to
have host updates from the `Discovery` service sent to the respective
ports that these services were already using because Netscript2 has a
bug in the implementation of `NetscriptPort.nextWrite` where you can
only subscribe to write events on one port per script, so trying to
use two separate ports would be more complex. Also, the throughput of
messages on these ports is low enough that it shouldn't be too much of
an issue. An alternative implementation of the subscription methods
that I'm still considering implementing is to batch sending all hosts
discovered

Please review this code, looking for possible bugs, particularly in
the way that messages are passed between them. Critique the design and
architecture of this distributed system, highlight the strengths and
weaknesses and offer any suggestions for improving it.



I just updated the structure of the host discovery service
(`src/services/discover.ts`), creating a pull API where other daemons
can send a request for the current list of workers or targets
available. This API also allows clients requesting these lists to
register a subscription to be notified of new hosts of that type. I
updated the `MemoryManager` service (`src/services/memory.tsx`), the
batch hacking `TargetSelector` service (`src/batch/manager.ts`) and
the batch hacking `Monitor` service (`src/batch/monitor.tsx`) to use
this new API and also to subscribe to newly discovered hosts of the
types that they care about.

I chose to have the `MemoryManager`, `TargetSelector` and `Monitor` to
have host updates from the `Discovery` service sent to the respective
ports that these services were already using because Netscript2 has a
bug in the implementation of `NetscriptPort.nextWrite` where you can
only subscribe to write events on one port per script, so trying to
use two separate ports would be more complex. Also, the throughput of
messages on these ports is low enough that it shouldn't be too much of
an issue. An alternative implementation of the subscription methods
that I'm still considering implementing is to batch sending all hosts
discovered

Please review this code, looking for possible bugs, particularly in
the way that messages are passed between them. Critique the design and
architecture of this distributed system, highlight the strengths and
weaknesses and offer any suggestions for improving it.
