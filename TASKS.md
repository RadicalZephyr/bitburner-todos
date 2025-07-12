# Task Written by Codex

1. Modify `src/batch/client/memory.ts`:
  * Extend `AllocationClaim` to include `hostname`, `filename`,
    `chunkSize`, and `numChunks`.
  * Update `registerAllocationOwnership` to collect these details from
    the running script (using `ns.self()`, `ns.getScriptRam()` if
    needed) and send them. Don't change the arguments to
    `registerAllocationOwnership`.

2.  In `src/batch/memory.tsx`:
  * Add a data structure to store multiple claims per `Allocation`
    (e.g., an array of `{pid, hostname, filename, chunkSize,
    numChunks}`).
  * Rewrite `claimAllocation` to record a claim in this structure
    instead of replacing a single `pid`.
  * Update `cleanupTerminated` to release only the portions owned by
    terminated PIDs and maintain the rest.
  * Adjust `releaseChunks` and `deallocate` to consider individual
    claims when freeing memory.

3. Update documentation in `docs/` to describe the new
   `AllocationClaim` format and manager behavior.

5. Run `npm run build` to ensure type-checking and compilation succeed.


# Additional work

1. Modify `src/batch/client/memory.ts`:
  * Extend `AllocationRelease` message to include the script pid and
    hostname.
  * Update `registerAllocationOwnership` to include the script pid and hostname.

2. Modify `src/batch/memory.tsx`:
  * Update the handling of `MessageType.Release` to account for the
    new information.
  * Adjust `releaseChunks` and `deallocate` to consider the `pid` and
    `hostname` when freeing memory.


# Plan Hacking Script

## Initial Plan

1. Update `src/batch/harvest.ts`:

   * Import `analyzeBatchThreads` from `"batch/expected_value"`.
   * In `main`, after target validation, compute thread counts using `analyzeBatchThreads(ns, target)`.
   * Determine RAM cost of the hack/grow/weaken scripts and sum them for `batchRam`.
   * Calculate `batchTime` using weaken time and `CONFIG.batchInterval`.
   * Compute `overlap` as `Math.ceil(batchTime / 1000)` and then `requiredRam = batchRam * overlap`.
   * Log these values so the player can see the needed memory.
2. Add any necessary doc comments.

This forms the basis for later expansion when launching batches continuously.


# Improve Script Logging

1. Update `src/batch/manage.ts`:

  * Change all calls to `ns.printf()` to use `ns.print()` and an
    interpolation string.
  * Add useful local data to all calls to `ns.print`.
  * Ensure the structure and formatting of all calls to `ns.print` is
    similar. Use prefixes to change the color as appropriate.
  * Add logging to important events that are missing it.

2. Update `src/batch/memory.tsx`:

  * Change all calls to `ns.printf()` to use `ns.print()` and an
    interpolation string.
  * Add useful local data to all calls to `ns.print`.
  * Ensure the structure and formatting of all calls to `ns.print` is
    similar. Use prefixes to change the color as appropriate.
  * Add logging to important events that are missing it.

Update `src/batch/memory.tsx`:

  * Add useful local data to all calls to `printLog`.
  * Ensure the structure and formatting of all calls to `printLog` is
    similar. Use prefixes to change the color as appropriate.
  * Add logging to important events that are missing it.

## Enhance the LogDisplay to colorize according to string prefix

Update `src/batch/memory.tsx`:

  * Enhance the `LogDisplay` React component to colorize each line of
    logs using the appropriate colors from the UI theme
    `ns.ui.getTheme()`. Make sure to handle the same logging
    colorizing as can be used by `ns.print()`.



# Plan improving MemoryManager

Help me plan a task for making the `MemoryManager` in
`src/batch/memory.tsx` track changes to scripts that were running
before it started. These might be scripts that were launched by the
user outside the `MemoryManager`. Currently, the `MemoryManager` only
checks for the total used RAM when it first receives a Worker.

Instead of just doing this check once, we should perform this check
once per second, not on every turn of the main loop.

When checking for untracked RAM usage we first need to check if the
memory we have allocated on that host is the same or greater than what
is actually in use.

If the memory in use is greater than what we have allocated we need
should then update the `reservedRam` on that worker with the
difference.

## The Suggested Task

1. **Add method to `MemoryManager`**\
   Implement `updateReserved()` that iterates over `workers.values()` and for each worker:

   ```ts
   const actual = ns.getServerUsedRam(worker.hostname);
   const diff = actual - worker.allocatedRam;
   worker.reservedRam = diff > 0 ? diff : 0;
   ```

   Include a short doc comment.

2. **Call the method once per second**\
   In the main loop of `main` (around the `cleanupTerminated` call), invoke `memoryManager.updateReserved()` within the same 1‑second check.

3. **Verify build**\
   Run `npm run build` and ensure there are no TypeScript errors.

4. **Commit**\
   Use commit message prefix `[batch]` per the contributor guide.


# Second try extending AllocationClaim

1. Modify `src/batch/client/memory.ts`:
  * Extend `AllocationClaim` to include `hostname`, `filename`,
    `chunkSize`, and `numChunks`.
  * Update `registerAllocationOwnership` to collect these details from
    the running script (using `ns.self()`, `ns.getScriptRam()` if
    needed) and send them. Don't change the arguments to
    `registerAllocationOwnership`.
  * Extend `AllocationRelease` message to include the script pid and
    hostname.
  * Update `registerAllocationOwnership` to include the script pid and
    hostname when sending `AllocationRelease`.
  * Expand logging statements to include new data.

2.  In `src/batch/memory.tsx`:
  * Add a data structure to store multiple claims per `Allocation`
    (e.g., an array of `{pid, hostname, filename, chunkSize,
    numChunks}`).
  * Rewrite `claimAllocation` to record a claim in this structure
    instead of replacing a single `pid`.
  * Update `cleanupTerminated` to release only the portions owned by
    terminated PIDs and maintain the rest.
  * Adjust `releaseChunks` and `deallocate` to consider individual
    claims when freeing memory.
  * Update the handling of `MessageType.Release` to account for the
    new information.
  * Adjust `releaseChunks` and `deallocate` to consider the `pid` and
    `hostname` when freeing memory.
  * Expand logging statements to include new data.
  * Add comments to complex logic explaining why the code is
    structured the way it is.

3. Update documentation in `docs/` to describe the new
   `AllocationClaim` format and manager behavior.

4. Run `npm run build` to ensure type-checking and compilation
   succeed. Fix any errors you find.

# Update batch-hacking.md to reflect the actual architecture

1. Read `src/start.ts`, `src/bootstrap.ts`, and the files in
   `src/batch`. Think about the ways that starting from the `start.ts`
   script they start running a distributed system that communicates
   through `NetscriptPort`s.

2. Update `docs/batch-hacking.org` to reflect the actual components we
   have and the important interactions between them.


# Implement Incomplete Contracts

1. Read `src/contracts/incomplete/Subarray-with-Maximum-Sum.ts`, pay special
attention to the block comment at the top of the file which describes
the problem to solve as well as some example input data and expected
output.

  * Break down the task into small achievable steps and create a
    `src/contracts/incomplete/TODO.md` with checkboxes for each step
    in your plan.

2. Modify `src/contracts/incomplete/Subarray-with-Maximum-Sum.ts`:

  * Reference the `src/contracts/incomplete/TODO.md` for the steps to
    solve the puzzle. Work through each task in the TODO file
    one-by-one implementing them. Make sure to check off each TODO
    list item when you finish it.

3. Read `src/contracts/incomplete/Unique-Paths-in-a-Grid-II.ts`, pay special
attention to the block comment at the top of the file which describes
the problem to solve as well as some example input data and expected
output.

  * Break down the task into small achievable steps in a new section
    in `src/contracts/incomplete/TODO.md` with checkboxes for each
    step in your plan.

4. Modify `src/contracts/incomplete/Unique-Paths-in-a-Grid-II.ts`:

  * Reference the `src/contracts/incomplete/TODO.md` for the steps to
    solve the puzzle. Work through each task in the TODO file
    one-by-one implementing them. Make sure to check off each TODO
    list item when you finish it.


# Add Partial Allocation Claim Test

1. Create a new file at `src/batch/client/tests/test_app.ts`:

  * Add the standard argument parsing boilerplate for accepting an
    `--allocation-id` flag and registering a claim of ownership of the
    passed allocation with `registerAllocationOwnership`.
  * Add a positional argument which should be a number indicating how
    long to `await ns.sleep()` before exiting.

2. Modify `src/batch/client/tests/mem_test_client.ts`:

  * At the end of the current script use the `MemoryClient` to request
    a new allocation of 300 chunks of 8 GB.
  * `ns.exec` several instances of
    `src/batch/client/tests/test_app.ts`.
  * Use the `--allocation-id` flag to transfer ownership of the
    allocation these processes.
  * Also pass them positional arguments of between 1000-3000.


# Solve Every Incomplete Contract

For every `*.ts` file in the directory `src/contracts/incomplete`:

1. Read the incomplete contract file you are working on:

  * Pay special attention to the block comment at the top of the file
    which describes the problem to solve as well as some example input
    data and expected output.

  * Break down the task into small achievable steps and create a
    `src/contracts/incomplete/TODO.md` with checkboxes for each step
    in your plan.

2. Modify the incomplete contract file you're working on:

  * Reference the `src/contracts/incomplete/TODO.md` for the steps to
    solve the puzzle. Work through each task in the TODO file
    one-by-one implementing them. Make sure to check off each TODO
    list item when you finish it.


# Complete batch/harvest script

1. Add an `autocomplete()` function that returns `data.servers`.
2. In `main()`, after computing logistics, request the needed RAM for
   all the overlapping batches `MemoryClient.requestTransferableAllocation(batchRam, overlap)`.
3. Develop a helper function that takes as arguments a
   `NS` object, a `TransferableAllocation`, and a `BatchLogistics`.

  * Loop through the `HostAllocation`s from
    `allocation.allocatedChunks`, add a new field to track how many
    batches have been spawned on this `hostname`.
  * This helper function should loop `logistics.overlap` times
    starting one batch each iteration.
  * Each batch should be spawned in this order:
    - a hack with args (`ns.exec("/batch/h.js, { threads: threads.hackThreads }, timings.hackStart, 0)`)

    - a weaken with args (`ns.exec("/batch/w.js, { threads: threads.postHackWeakenThreads }, timings.postHackWeakenStart,0)`)

    - a grow with args (`ns.exec("/batch/g.js, { threads: threads.growThreads }, timings.growStart, 0)`)

    - a weaken with args (`ns.exec("/batch/w.js, { threads: threads.postGrowWeakenThreads }, timings.postGrowWeakenStart, 0)`)

  * Start a new batch after waiting (`await ns.sleep(CONFIG.batchInterval)`), keeping track of running batches.

  * Keep track of the pids of running batches and the hostname

6. Include doc comments for any new exported functions and ensure the script loops indefinitely until killed.
7. Run `npm run build` and fix any TypeScript errors.


Use `src/batch/sow.ts` and `src/batch/launch.ts` as implementation references.

# Develop custom harvest launch function

Modify `src/batch/harvest.ts`:

1. Add a helper function `launchBatch(ns: NS, allocation:
   TransferableAllocation, logistics: BatchLogistics)`

2. The body of the function should loop through allocation
  * Loop through the `HostAllocation`s from
    `allocation.allocatedChunks`, add a new field to track how many
    batches have been spawned on this `hostname`.
  * This helper function should loop `logistics.overlap` times
    starting one batch each iteration.
  * Each batch should be spawned in this order:
    - a hack with args (`ns.exec("/batch/h.js, { threads: threads.hackThreads }, timings.hackStart, 0)`)

    - a weaken with args (`ns.exec("/batch/w.js, { threads: threads.postHackWeakenThreads }, timings.postHackWeakenStart,0)`)

    - a grow with args (`ns.exec("/batch/g.js, { threads: threads.growThreads }, timings.growStart, 0)`)

    - a weaken with args (`ns.exec("/batch/w.js, { threads: threads.postGrowWeakenThreads }, timings.postGrowWeakenStart, 0)`)

  * Start a new batch after waiting (`await ns.sleep(CONFIG.batchInterval)`), keeping track of running batches.

  * Keep track of the pids of running batches and the hostname


# Extend Find-Largest-Prime-Factor

Modify `src/contracts/Find-Largest-Prime-Factor.ts`:

1. Implement a helper function for finding prime numbers.

2. Replace `factors.push(n)` with usage of the prime finding helper
   function to find all the prime factors up to the square root of `n`
   and keep dividing them out and pushing them into the `factors`
   array.

# Add option to prioritize home usage for core-dependent functions

Modify `src/batch/client.ts`:

1. Add an optional field `coreDependent` to `AllocationRequest`.

2. Add an argument to `requestTransferableAllocation` to fill this new
   field, defaulting it to false if not specified.

Modify `src/batch/memory.tsx`:

1.

## Meta Task

Some tasks benefit substantially from being run specifically on the
"home" server if the player has purchased additional cores for it. To
update the `MemoryManager` to handle this I want to add a new option
to the memory handling `AllocationRequest` to allow `MemoryClient`
users to specify that their workload is "core dependent". The
`MemoryManager` should then prefer to allocate memory for these
`coreDependent` tasks from the "home" server. It should also prefer to
allocate tasks that are not `coreDependent` on servers other than the
"home" server.

The `MemoryManager` is located in the file `src/batch/memory.tsx` and
the `MemoryClient` is located in the file
`src/batch/client/memory.ts`.

Write a task definition to accomplish this enhancement to the memory
handling subsystem.

### Followup

Modify `src/batch/launch.ts`:

Add an option to the `LaunchRunOptions` to specify that this workload
is `coreDependent` and pass this option to
`requestTransferableAllocation`.

Add a command line flag to allow usage of this option as well.


# Make MemoryDisplay and LogDisplay independently scrollable

Currently the logging output in the `src/batch/memory.tsx` file has
only a single scroll control for both columns of it's output, the
`MemoryDisplay` and the `LogDisplay`.

We cannot change the styles associated with the parent element of
these displays.

Make it so that the user can scroll the content of the `MemoryDisplay`
and the `LogDisplay` separately from each other.



# Potential Future Tasks

## Create a communication flow between the MemoryManager and the TaskSelectionManager

Currently, the TaskSelectionManager starts tasks with


## Fix bug in Harvest Script

There is a bug in the memory allocator. Some scripts eventually stop
spawning new batches. Their log files all contain a specific number of
messages like this:

```
INFO: claiming allocation 34 pid=12402758 host=pserv32TB 1x10.70GiB batch/harvest.js
johnson-ortho: batch ram 6.95GiB, overlap x364 => required 2.47TiB
phases: [
  {
    "script": "h.js",
    "start": 272166.16353401105,
    "duration": 90763.72117800369,
    "threads": 1
  },
  {
    "script": "w.js",
    "start": 0,
    "duration": 363054.88471201475,
    "threads": 1
  },
  {
    "script": "g.js",
    "start": 72735.97694240295,
    "duration": 290443.9077696118,
    "threads": 2
  },
  {
    "script": "w.js",
    "start": 250,
    "duration": 363054.88471201475,
    "threads": 0
  }
]
INFO: requesting 364 x 6.95GiB contiguous=false coreDependent=false
SUCCESS: allocated id 35 on 1 hosts
INFO: claiming allocation 35 pid=12402758 host=pserv32TB 1x10.70GiB batch/harvest.js
WARN: failed to spawn /batch/w.js on pserv32TB
WARN: failed to spawn /batch/w.js on pserv32TB
WARN: failed to spawn /batch/h.js on pserv32TB
```

Read `src/batch/harvest.ts`.


## Look for bugs in the memory handling subsystem

Read `src/batch/memory.tsx`, and
`src/batch/client/memory.ts`. Create a file `MEMORY_BUGS.md` and fully
document


## Expose total free memory

Add a new message type `Status` in `src/batch/client/memory.ts`.\
Implement corresponding handling in `src/batch/memory.tsx` that writes `{freeRam: number}` back to the requesting port.\
Create `MemoryClient.getFreeRam(): Promise<number>` that sends the status request and resolves to the returned `freeRam`.


## Buy Hacknet Script

1. Add new file `src/buy-hacknet.ts`.
2. Use `ns.flags()` to parse:

   * `--return-time` (minutes, default 60).
   * `--spend` (0–1, default 1).
   * `--help`.
3. When `--help` is passed or parameters are invalid, print usage information with `ns.tprint`.
4. Determine allowed budget: `ns.getServerMoneyAvailable("home") * options.spend`.
5. Gather stats of existing nodes through `ns.hacknet` APIs.
6. Use `ns.formulas.hacknetNodes.moneyGainRate` (from `NetScriptDefinitions.d.ts`) to predict income from upgrades or new nodes.
7. Decide purchases and upgrades whose predicted payback time is ≤ `return-time` minutes and whose cost fits in the budget.
8. Perform the purchases with the relevant `ns.hacknet` methods (`purchaseNode`, `upgradeLevel`, `upgradeRam`, `upgradeCore`).
9. Print informative logs using `ns.print`, formatting numbers/time with `ns.formatNumber`, `ns.tFormat`, etc., following the style in `AGENTS.md`.
10. Export only `main` (no doc comments required per contributor guide).
11. Run `npm run build` and ensure no type errors.

Commit message prefix: `[buy-hacknet]`.


##

## Put Targets in correct Pending List on add

In `src/batch/manage.ts`, instantiate a `MemoryClient`.\
During each tick, call `getFreeRam()` to obtain the current free RAM.\
Use functions from `till.ts`, `sow.ts`, and `harvest.ts` to estimate RAM required for each pending task and sort them by `expectedValuePerRamSecond`.\
Launch tasks greedily while enough RAM remains, passing the calculated
thread counts to each script.

Modify `src/batch/manage.ts`:
1. Update `TargetSelectionManager.pushTarget()`:

   * Retrieve `minSec`, `curSec`, `maxMoney`, and `curMoney` using Netscript API.
   * If `curSec > minSec + 1` → log with `ns.print`, push the host to
     `pendingTargets` (tilling queue) and call `monitor.pending`.
   * Else if `curSec <= minSec + 1` and `curMoney < maxMoney * 0.999` → log with `ns.print` and push to `pendingSowTargets`.
   * Else if `curSec <= minSec + 1 && curMoney >= maxMoney * 0.999`  →
     log with `ns.print` and push to `pendingHarvestTargets` and call `monitor.pendingHarvesting`.
2. Retain the `allTargets` set check to avoid duplicates.
3. Add a doc comment describing the method’s new behavior.
4. Ensure logging uses the color-coded prefixes from `AGENTS.md`.
5. Run `npm run build` to verify compilation.


## Meta Task - How to improve batch hacking script to scale better

The batch hacking system is in a pretty good place for starting the
next phase of improvement. One feature of Bitburner as an idle game is
resetting some of your current progress to gain permanent long-term
improvements that let you progress further. In Bitburner this takes
the form of augmentations that improve things like the rate you gain
experience in different skills, the speed with which hack weaken and
grow complete, how effective those functions are etc.

There are two main things that change when you reset progress to
install new augmentations. You lose any files you've purchased and you
lose all of your current money and skill progress. So every time you
install augmentations you start over with very little money and much
less overall memory available to run scripts.  Both of these things
will change as we start a run by immediately beginning the batch
hacking scripts, so


## Create a leak-check.ts

1. **Extend `MemoryClient`**

   * File: `src/batch/client/memory.ts`
   * Add `Snapshot` (or similar) to `MessageType`.
   * Create interfaces describing a snapshot:

     * `WorkerSnapshot` with worker RAM properties.
     * `AllocationSnapshot` summarizing allocations and claims.
     * `MemorySnapshot` containing arrays of workers and allocations.
   * Implement `memorySnapshot(): Promise<MemorySnapshot>` sending a new\
     `Snapshot` message and reading the response from a private return port.
   * Document the new method with a JSDoc comment.

2. **Handle snapshot requests in MemoryManager**

   * File: `src/batch/memory.tsx`
   * Add a case for the new message type in `readMemRequestsFromPort`.\
     Call a `getSnapshot()` method on `MemoryManager` and write the result\
     to the requesting port.
   * Implement `getSnapshot()` in the `MemoryManager` class that builds\
     the `MemorySnapshot` structure from current workers and allocations.

3. **Create `src/batch/leak-check.ts`**

   * Implement a script that uses `MemoryClient.memorySnapshot()` to fetch the\
     snapshot.
   * Validate for inconsistencies:

     * workers’ used RAM should never exceed total RAM.
     * each allocation’s claims must not exceed reserved chunks.
     * totals reported by workers should match allocation totals.
   * Report any problems via `ns.print`.

4. **Run `npm run build` to ensure compilation succeeds.**

This will provide detailed visibility into memory allocations and a tool to\
detect leaks or mismatched accounting.


## Change RAM Calculations to Fixed-Point

1. Add utility functions in `src/batch/memory.tsx`:

   ```ts
   const toFixed = (val: number): bigint => BigInt(Math.round(val * 100));
   const fromFixed = (val: bigint): number => Number(val) / 100;
   ```
2. Change `setAsideRam`, `reservedRam`, and `allocatedRam` in the `Worker` class to type `bigint`.

   * Convert incoming numbers with `toFixed` in the constructor.
3. Update `Worker` methods:

   * `usedRam` and `freeRam` should return numbers using `fromFixed`.
   * `allocate()` and `free()` must adjust `allocatedRam` using `toFixed`.
4. Update `MemoryManager.updateReserved()` to compute and store reserved RAM as `bigint`.
5. In `getSnapshot()` convert all worker RAM fields back to numbers using `fromFixed`.
6. Adjust UI helpers (`MemoryBar`, etc.) to call `fromFixed` when reading worker RAM values.
7. Run `npm run build` and fix any TypeScript errors introduced by the new types.


## Plan Task for dealing with miscalculated batches

Let's talk about how to handle when the target server gets out of


## Rebalance Batches

1. Implement `calculateRebalanceBatchLogistics(ns, target, batchRam)` in `src/batch/harvest.ts`.

   * Prioritize weaken threads to reach `minSecurity`.
   * Determine grow threads needed to return to `maxMoney`.
   * Ensure the computed `rebalanceBatch.batchRam` (or `ramSize`) is ≤ `batchRam`.

2. In the main harvest loop, check after each batch whether security
   exceeds `minSecurity` or money is below `maxMoney`.

   * When deviation is detected, use `calculateRebalanceBatchLogistics` to compute phases and spawn a rebalance batch using the existing `spawnBatch` helper.
   * Only spawn this batch if its RAM fits within `batchRam`.

3. Include doc comments for the new exported function and apply formatting helpers (`ns.formatRam`, `ns.formatPercent`) when logging.

4. Verify `npm run build` succeeds without type errors.


## Add Heartbeat to Manager

1. **Extend client message definitions**

   * In `src/batch/client/manage.ts` introduce
     `MessageType.Heartbeat`.
   * In `src/batch/client/manage.ts` add a new `Lifecycle` enum with
     `Till`, `Sow` and `Harvest` variants.
   * Add a `Heartbeat` interface with `pid`, `filename`, `target`, and `lifecycle`
   * Update `Payload` to `string | Heartbeat` and adjust `sendMessage` to accept this union.
   * Implement `heartbeat()` on `ManagerClient` that sends a `Heartbeat` message. Include a doc comment.

2. **Handle heartbeats in the manager**

   * Update `readHostsFromPort` in `src/batch/manage.ts` to recognize `MessageType.Heartbeat` and forward the payload to the manager.
   * Add a method `handleHeartbeat(hb: Heartbeat)` on `TargetSelectionManager`.\
     Ensure the target appears in the correct internal list (`tillTargets`, `sowTargets`, or `harvestTargets`) based on `hb.lifecycle`, removing it from pending arrays if necessary.

3. **Send heartbeats from worker scripts**

   * Import `Lifecycle` and `ManagerClient` in `src/batch/harvest.ts`, `src/batch/sow.ts`, and `src/batch/till.ts`.
   * Instantiate a `ManagerClient` and, once per second during each script’s main loop, call `heartbeat(ns.pid, ns.getScriptName(), target, Lifecycle.<appropriate>)`.

4. **Verify compilation**

   * Run `npm run build` and resolve any TypeScript errors.

This will let the manager recover state when restarted and keep its target lists synchronized with the actual running processes.


## Use HackingMultipliers to calculate threads

1. In `src/batch/expected_value.ts`:

   * Fetch multipliers with `const mults = ns.getHackingMultipliers();`.
   * In `successfulHackValue`, multiply the calculated steal amount by `mults.money`.
   * In `growthAnalyze`, divide the thread count by `mults.growth`\
     (after formulas or `ns.growthAnalyze` result).
2. In `src/batch/harvest.ts` function `hackThreadsForPercent`,\
   multiply `hackPercent` by `mults.money` before computing the thread count.
3. Apply the same adjustment to any helper functions (e.g. `growthAnalyze` in `sow.ts`) that compute grow threads.
4. Ensure all exported functions have updated doc comments.
5. Run `npm run build` and fix any type errors.


## Discuss More Restartable Structure for Daemons

Let's talk about making each of the long-running service daemons in
`src/services/memory.tsx`, `src/services/discover.ts`,
`src/batch/manage.ts` and `src/batch/monitor.tsx` more robust to
restart individually.  Currently several of them depend on important
information being sent to them during the startup sequence. For
instance, since the Discovery service explicitly pushes workers to
`MemoryManager` and targets to `TargetSelectionManager` it is
implicitly assuming that it can send each host to each service once
when it is first discovered. This assumption is obviously violated if
the `MemoryManager` or `TargetSelectionManager` is restarted without
the `Discovery` service being restarted.

Similarly, the `Monitor` daemon depends on receiving messages when the
`TargetSelectionManager` decides a target should change phase, but if
the `Monitor` is restarted it has no way to receive the current
status.

I think overall, we need to shift from a push-based strategy for
communication to a pull-based one.

## New `DiscoveryClient`

Write a task for adding a `DiscoveryClient` in a new source file
`src/services/client/discovery.ts`. Instead of every client choosing a
unique port to receive their response on, the `DiscoveryClient` will
use a constant response port number on which all client responses will
be multiplexed. Clients will instead compute a client ID (`clientId =
makeRequestId(ns)`) and send that to the Discovery server. The server
will send these `clientID`s back with the response to allow the client
to identify their response. Use this helper function
(`makeRequestId()`) to generate the IDs.

```typescript
/** Generate a simple unique ID: pid‐timestamp‐rand **/
export function makeRequestId(ns) {
  const pid = ns.self().pid;
  const ts  = Date.now();
  const r   = Math.floor(Math.random() * 1e6);
  return `${pid}-${ts}-${r}`;
}
```

The Discovery server will be rewritten substantially. Instead of
proactively sending messages to  on it's
request port and sending responses.

There will be two types of requests for the Discovery server `workers`
and `targets`. Both will send back a list of hostnames. The `workers`
will only send hosts that have `ns.


## Stock Tracker/Trader Roadmap Implementation

Implement the architecture detailed according to the Development
Roadmap specified. Make a commit for each phase of the Roadmap.


## Scale down batches in manager

- Allow manager to calculate small enough batches to spawn them.


## Change MemoryManager to be more flexible

When memory is scarce and we get new requests either:
- give partial allocations (probably bad)
- maintain a priority queue of memory allocation requests and allow
  client to wait until it's allocation is satisfied?


Figure out why "home" server is sooo underutilized. Probably something
to do with the sorting?


## Simpler hacking method for bootstrapping

Using the whole batching system just takes too much memory I think,
when starting a new Bitnode.

Options:

- Use naive h/g/w script?
- Use loop hacking scripts?


# New Port Allocator Service

1. Create `src/services/port.tsx` with a `PortAllocator` daemon.
   * Maintain a set of currently allocated port numbers.
   * On `PortRequest`, return an unused port; on `PortRelease`, clear
     it (`ns.clearPort(id)`) and remove it from the set.
   * Only manages port ids larger than 100
   * Ports for discovery, memory, etc. should remain reserved and not offered for allocation.
2. Add `src/services/client/port.ts` implementing a client API.

   * Define `PORT_ALLOCATOR_PORT` and `PORT_ALLOCATOR_RESPONSE_PORT`.
   * Enumerate `MessageType` with at least `PortRequest` and `PortRelease`.
   * Extend `Client` from `src/util/client.ts` for sending and receiving messages.
3. Update any scripts needing dynamic ports (e.g. `batch/harvest.ts`) to request a port via this client instead of using `ns.pid`.
4. Ensure all exported functions have doc comments.
5. Run `npm run build` and `npx jest` to verify typings and tests.


## Growable Allocations

1. In `src/services/memory.tsx`, update the request loop to handle `GrowableRequest`.

   * Allocate memory as with `MessageType.Request`.
   * Store the request’s port on the created allocation so extra
     chunks can be sent later.

2. In `src/services/allocator.ts`, modify `Allocation` to track:

   * desired total chunks (`requestedChunks`) and an optional `notifyPort`.

3. Add a method `growAllocation(allocation: Allocation, numChunks: number)` that allocates additional chunks if workers have free RAM.

4. Periodically check for free RAM in the main loop (`src/services/memory.tsx`) and attempt to grow allocations toward `requestedChunks`.

   * When new chunks are added, send a message over the stored `notifyPort` describing the host and chunk information.

## Implement GrowableAllocation

1. In `src/batch/client/growable_memory.ts`, define `GrowableAllocation`.

   * Store the allocation id, current chunks, and the listening port.
   * Continuously poll the port for new chunk messages and merge them into the internal chunk list.
2. Provide a method `launch(ns: NS, script: string, threads: number | LaunchRunOptions, ...args: ScriptArg[])`.

   * Mirror `launch` from `src/services/launch.ts` but distribute threads across the internal chunks.
   * Handle dependency copying as done in the original `launch` helper.
3. Provide `release(ns: NS)` and `releaseAtExit(ns: NS)` similar to `TransferableAllocation`.

## Test Growable Allocations

The client interface to creating `GrowableAllocation`s is in
`src/services/client/growable_memory.ts`.

The `MemoryAllocator` is in `src/services/allocator.ts` and the
message handling is in `src/services/memory.tsx`.

I want to create a new client test program, similar to the file at
`src/services/tests/mem_test_client.ts` that uses the
`GrowableMemoryClient` to request a `GrowableAllocation`.

It should start by first allocating all but 8GB of available memory
(`MemoryClient.getFreeRam()`) in 8GB chunks in a
`TransferableAllocation`.

It will then request a `GrowableAllocation` of 32x8GB
chunks. Initially, only 1 chunk should be returned, and we should
verify this.

Then once every five seconds the script will release one 8GB chunk
from the `TransferableAllocation` using `MemoryClient.releaseChunks`,
wait for 1 second and verify that the `GrowableAllocation` has
increased by one chunk.
