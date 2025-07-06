# Version 1

1. **Add `src/batch/test.ts`**

   * Accept a target server as the first positional argument.
   * Support optional flags:

     * `--iterations` (default ~5) – number of trials per configuration
     * `--max-threads` (upper bound for thread-count sweep)
     * `--formulas` (use `ns.formulas` when available)
     * `--use-mults` (apply `ns.getHackingMultipliers()`; otherwise treat multipliers as 1)
   * Obtain one contiguous memory allocation via `MemoryClient.requestTransferableAllocation`.

2. **Testing loop**

   * For each thread count from 1 to `maxThreads`:

     * Reset the target to max money and min security using existing `till`/`sow` helpers.
     * Compute predicted money and security deltas for hack and grow using both basic Netscript functions and formulas (according to flags).
     * Spawn `/batch/h.js` or `/batch/g.js` with the selected thread count.
     * After completion, record actual money change and security change.
     * Log prediction vs. actual differences using `ns.print` with `INFO:`/`SUCCESS:` prefixes.

3. **Result reporting**

   * Average results over `iterations` for each combination.
   * Print a summary table showing prediction error per algorithm and thread count.

4. **Documentation and build**

   * Provide doc comments for any exported helpers.
   * Ensure grow/weaken thread calculations use `Math.ceil` per `src/batch/AGENTS.md`.
   * Run `npm run build` and fix any TypeScript errors.
   * Commit with message prefix `[batch]`.

This experiment script will make it easy to evaluate hack/grow/weaken
calculations under different conditions, enabling refinement of
prediction algorithms.


# Version 2

1. **Create** `src/batch/test.ts`.
2. Accept a target host as positional arg.\
   Optional flags:\
   `--iterations` (default 5), `--max-threads` (default 1), `--use-formulas` (boolean), `--apply-multipliers` (boolean).
3. Request a contiguous chunk from `MemoryClient.requestTransferableAllocation` for helper scripts.
4. For each thread count from 1 up to `--max-threads`:

   * Calculate predicted hack/grow effects using:

     * Built‑in analysis.
     * Formulas API (when `--use-formulas` is true).
     * Both with and without applying `ns.getHackingMultipliers()` when `--apply-multipliers` is toggled.
   * Execute one thread (or the specified count) of `/batch/h.js` or `/batch/g.js`.
   * Measure actual money/security changes.
   * Log `INFO:` lines showing predicted vs actual and the error.
   * Repeat for the number of iterations to average the error.
5. Reset the server between iterations using existing `till` and `sow` helpers to ensure min security and max money.
6. Release the allocated memory on exit.
7. Add doc comments for any exported helpers and follow existing logging prefixes.

Check that `npm run build` succeeds without type errors. Commit using a message starting with `[batch]`.


# Version 3

Create `src/batch/test.ts` to evaluate prediction accuracy for hack/grow cycles.

1. Accept a target hostname as a positional argument and optional flags for:

   * iteration count
   * max threads to test (e.g., 1–10)
   * calculation mode: basic API, formulas API, with/without `ns.getHackingMultipliers()`.

2. Request a single contiguous allocation from `MemoryClient.requestTransferableAllocation` to launch helper scripts (`/batch/h.js`, `/batch/g.js`, `/batch/w.js`). Release it when done.

3. For each thread count (1..N) and each calculation mode:

   * Use helpers from `src/batch/expected_value.ts` to compute predicted money and security effects of hack and grow.
   * Execute the corresponding script once, measure actual money/security change, and log the difference.
   * Reset the target to min security and max money using existing `till`/`sow` helpers between iterations.

4. Average errors across iterations and print a summary with the log formatting guidelines from `AGENTS.md` (e.g., “INFO: ” or “WARN: ”).

5. Add doc comments for all exported functions and ensure all numerical thread calculations return integers as instructed in `src/batch/AGENTS.md`.

6. Verify the build with `npm run build` and fix any type errors.

7. Commit with a message like `[batch] add experimental prediction test script`.


# Version 4

1. Implement `src/batch/test.ts`:

   * Uses `ns.flags()` to parse:

     * target hostname (positional).
     * `--iterations` (default 1).
     * `--max-threads` (upper bound for thread-count sweep).
     * `--use-formulas` (boolean).
     * `--use-mults` (boolean to include `ns.getHackingMultipliers()` in predictions).
   * Request a single contiguous chunk from `MemoryClient`.
2. For each thread count from 1 to `maxThreads`:

   * Predict hack and grow results using both basic and formulas APIs, toggling multipliers according to flags.
   * Spawn the appropriate `/batch/h.js`, `/batch/g.js`, and `/batch/w.js` helpers with that thread count.
   * Record money/security before and after each script and compute errors.
   * Repeat for `iterations` runs to average the results.
3. Log all results with the standard `INFO:`/`SUCCESS:`/`WARN:` prefixes and formatted numbers as directed in `AGENTS.md`.
4. Document the script’s usage in `docs` or README.
5. Verify the build with `npm run build` and commit with message prefix `[batch]`.

This script will enable automated experimentation across thread counts
and calculation modes, revealing how closely each approach matches the
in-game mechanics.


## Combined

1. **Create** `src/batch/test.ts`.
2. Accept a target host as positional arg.\
   Optional flags:\
   `--iterations` (default 5), `--max-threads` (default 1), `--use-formulas` (boolean), `--apply-multipliers` (boolean).
3. Request a contiguous chunk from `MemoryClient.requestTransferableAllocation` for helper scripts.
4. For each thread count from 1 up to `--max-threads`:

   * Calculate predicted hack, and grow effects using:

     * Built‑in analysis.
     * Formulas API (when `--use-formulas` is true).
     * Both with and without applying `ns.getHackingMultipliers()` when `--apply-multipliers` is toggled.
   * Execute one thread (or the specified count) of `/batch/h.js` or `/batch/g.js`.
   * Measure actual money/security changes.
   * Write the recorded data in the csv format to a file for each
     function being measured (hack, grow and weaken) using
     `ns.write("resultsHack.txt", data, "a")`. Record the number of
     threads, the actual measured changes in security and money, and
     the predicted change in value money and security, for each
     prediction method.
   * Repeat for the number of iterations to average the error.
5. Reset the server between iterations using existing `till` and `sow` helpers to ensure min security and max money.
6. Release the allocated memory on exit.
7. Add doc comments for any exported helpers and follow existing logging prefixes.
8. Document the script’s usage in it's `--help` USAGE message.

Check that `npm run build` succeeds without type errors. Commit using a message starting with `[batch]`.
