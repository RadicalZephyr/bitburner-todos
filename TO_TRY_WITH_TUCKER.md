# Ideas of tasks to try with Tucker

## Fill out the README with better "how to use this repo" instructions

Include fixing the alias for "bootstrapping" the repo by renaming the
bootstrap script that is built by Github Actions.

## Find the most file with the most complex code

Suggest a refactoring to make it more readable and testable.

More of a demo task TBH, but it could have interesting results.

## Meta Task for coming up with interesting tasks

Try to construct a meta-prompt telling it to come up with off-the-wall
ideas for tasks to ask Codex to do.

## Add MEMORY_TAG to every scripts flags array

Make it so every script that has a main function includes the
`MEMORY_TAG` flag so the new launch service can tag it.

## Add allocation ID flag to every script

Add the `allocation-id` flag to every script and make it fully
standardized, and remove the option from the `launch` script to
specify a different allocation flag.

## Trim RAM Bloat from task selector and monitor file

Explain how each `ns` function in the file or any imported file costs
RAM whether it's used or not. Ask it to examine the imports of the
task selector and monitor files and ensure that every RAM function the
script will be charged for is actually being called. If there are
unused functions, break up any imported files so that the minimum
necessary number of `ns` calls will be attributed to each script.

## New Launch Service

See HUMAN_TODO.org for details.

## Refactoring harvest main function into more modular (and testable?) chunks

It's gotten very long and quite intricate, we could ask Codex to try
and break down the complexity and extract out well-named conceptually
coherent functions to make it cleaner and easier to read.

## Refactoring parameter lists into options objects

I'm pretty confident it can do it, so not super interesting.
