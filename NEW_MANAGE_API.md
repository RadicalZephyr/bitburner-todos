# Version 1

1. Extend **`src/batch/client/manage.ts`**

   * Add a new enum entry (e.g. `RequestLifecycles`) in `MessageType`.
   * Define a response type such as `Record<string, Lifecycle>` or `[string, Lifecycle][]`.
   * Implement `requestLifecycles()` in `ManagerClient` using `sendMessageReceiveResponse`.

2. Update **`src/batch/manage.ts`**

   * Obtain `MANAGER_RESPONSE_PORT` and pass it to a modified `readHostsFromPort`.
   * Add logic in `readHostsFromPort` to detect the new `RequestLifecycles` message, call a new method on `TargetSelector` to gather current states, and write the response `[requestId, payload]` to the response port.
   * Implement `TargetSelector.getLifecycleMap()` that returns the lifecycle phase for all known targets using the existing sets (`tillTargets`, `sowTargets`, etc.).

3. Modify **`src/batch/monitor.tsx`**

   * Instantiate `ManagerClient` in `main`.
   * After initial Discovery requests, call `requestLifecycles()` to get the current state map and seed `lifecycleByHost`.
   * Handle failure gracefully (if manager is absent).

4. Ensure every exported function gains doc comments and adjust any thread calculation functions to return integers per `src/batch/AGENTS.md`.

5. Run `npm run build` and `npx jest` to verify compile and test status.

6. Commit with message prefix `[batch]` as per repository guidelines.

# Version 2

1. **Extend message types**

   * File: `src/batch/client/manage.ts`
   * Add a new `MessageType` (e.g., `RequestLifecycle`).
   * Define an appropriate request/response payload structure mapping
     hostnames to `Lifecycle` values such as `[string, Lifecycle][]`.
   * Implement a new `requestLifecycle()` method in `ManagerClient` that returns the snapshot via `sendMessageReceiveResponse`.
   * Document the new method with a doc comment.

2. **Handle requests in the manager**

   * File: `src/batch/manage.ts`
   * Update `readHostsFromPort()` to recognize `MessageType.QueryLifecycle`.
   * Implement logic to collect lifecycle information from the `TargetSelector` sets (`pending*`, `tilling`, `sowing`, `harvestTargets`, etc.) and send the response to `MANAGER_RESPONSE_PORT`.
   * Ensure all returned values use the same lifecycle enum.

3. **Expose TargetSelector snapshot helper**

   * File: `src/batch/manage.ts`
   * Add a method on `TargetSelector` (e.g., `snapshotLifecycle()`).

     * Returns a `[string, Lifecycle]` for every tracked host.
     * Used by the new request handler.
   * Include a doc comment describing the snapshot format.

4. **Update Monitor to pull on startup**

   * File: `src/batch/monitor.ts`
   * Instantiate `ManagerClient` and call `requestLifecycle()` when the monitor starts.
   * Populate `lifecycleByHost` with the returned map before listening for pushed updates.

5. **Verify heartbeat sources**

   * Files: `src/batch/till.ts`, `src/batch/sow.ts`, `src/batch/harvest.ts`
   * Ensure these scripts already send regular heartbeats via `ManagerClient.heartbeat`. No functional change required, but confirm calls remain correct after enum updates.

6. **Run tests and build**

   * Execute `npm run build` and `npx jest` to ensure the codebase builds and tests pass.
   * Fix any TypeScript or test issues introduced by the new API.
