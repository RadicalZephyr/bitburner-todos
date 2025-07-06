# Improve the `TaskSelector` (`src/batch/task_selector.ts`) when RAM is highly constrained

Currently, the code does not do a good job scaling down operations
when relatively small amounts of RAM are available. This is a rough
outline of our plan to improve the `TaskSelector`.

## PID tracking and timeouts

Store the PID (and script type) for each launched worker. After a
configurable timeout (e.g., a few seconds), if no heartbeat has been
received and ns.isRunning(pid) returns false, consider the launch
failed and requeue the target. This prevents repeated attempts without
progress.

Each target should only ever be targeted by one hacking task script
and when launching a new task we only ever launch a single thread so
there should never be more than one PID returned from launching the
task.

We'll use a Heartbeat timeout of three seconds before considering a task
to have failed. During this waiting period we won't launch any new
tasks to give the new tasks time to request memory for their helpers.

## Back‑off strategy

When a new task we spawn fails due to not being able to allocate RAM
for it's helpers the `TaskSelector` will slow down the global rate of
launching new tasks. We will use an exponential backoff strategy with
a maximum of ten seconds between trying to launch new tasks.

The backoff needs to apply only to attempting to launch new tasks,
processing incoming messages shouldn't be slowed down.

## Handling unexpected termination

A running task script may crash or be killed, leaving the target
stuck. Track the last heartbeat timestamp per PID and drop the target
back to the pending list if the heartbeat has not been updated for
several seconds.

## Estimating RAM requirements

The manager currently deducts only the small amount of RAM needed for
the wrapper script, but the task scripts allocate additional RAM for their
helper scripts. Tracking the approximate RAM footprint of each
operation (e.g., expected threads from
calculateSowThreads/calculateWeakenThreads) allows the manager to make
a more realistic decision about whether another task can be started.

## Recovery on manager restart

Because task scripts send periodic heartbeats, the manager can rebuild
its internal state when restarted from the heartbeats it receives from
. Make sure the first heartbeat includes enough information (PID,
type, target) so the manager can restore the pending/running sets
correctly.

Ask me questions about the most important unspecified details to help
clarify this plan and make it more comprehensive.


# Clarifying details

## PID tracking

Each target should only ever be targeted by one hacking task script
and when launching a new task we only ever launch a single thread so
there should never be more than one PID returned from launching the
task.

We'll use a Heartbeat timeout of three seconds before considering a task
to have failed. During this waiting period we won't launch any new
tasks to give the new tasks time to request memory for their helpers.

## Backoff strategy

When a new task we spawn fails due to not being able to allocate RAM
for it's helpers the `TaskSelector` will slow down the global rate of
launching new tasks. We will use an exponential backoff strategy with
a maximum of ten seconds between trying to launch new tasks.

The backoff needs to apply only to attempting to launch new tasks,
processing incoming messages shouldn't be slowed down.


# Estimating RAM requirements

We already estimate the ram requirements for a full batch, but we want
to scale down the operation of the system because RAM is constrained,


# Task

Store the PID returned by `launch()` for each launched script in a
`pendingLaunches` map. When a `Heartbeat` is received with that PID,
move the host to the corresponding active set. If no heartbeat arrives
within ~2 s, treat the launch as failed and requeue the target.\
Modify `TaskSelector.handleHeartbeat` and the launch methods in
`src/batch/task_selector.ts`.
