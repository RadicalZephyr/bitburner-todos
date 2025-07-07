# Smooth Batch Hacking Task Rebalancing

I have two problems that are related with my current batch hacking
system in my Bitburner scripts. Currently my `TaskSelector`
(`src/batch/task_selector.ts`) uses a very simple greedy algorithm
when choosing tasks, it first tries to launch the highest expected
value harvest task, then it tries to launch a sow task against the
lowest level target, then it tries to launch a till task against the
lowest level target.

This works well for choosing the most valuable tasks currently
available, and it performs well starting from the beginning of a
Bitnode when total RAM available to launch scripts is low and our
hacking level is low. However, as these two factors change, the
initial greedy selection becomes less and less optimal, and there is
currently no mechanism for reassessing what tasks are running and
whether they are still worth while.

To put it in perspective, the spread of earning potential between
targets once we get to a decently high hacking level can be very
large. Some early game targets can at most be harvested for a few
million dollars per second, whereas some end stage targets can be
harvested for nearly $500 billion per second.

Another moment when the greedy algorithm falls down is when a lot of
new RAM becomes available, such as when we acquire a new port opening
program which allows us to nuke a whole new set of hosts on the
network. A similar situation occurs when we buy personal servers for
the first time. When lots of new RAM becomes available, the current
greedy algorithm is only able to select new tasks. This fails
particularly badly early in a bitnode when it is likely that we will
have spawned a scaled-down harvest task against one of the early game
targets like n00dles or foodnstuff. If we get more memory in this
situation it is _always_ going to be more valuable to scale up the
current harvest task to full capacity than to start moving a new
target through the hacking lifecycle.

## Another use-case to consider

While thinking about solutions to this issue of how to
reprioritize tasks and scaling up existing jobs, I also want to
consider how to handle upgrading personal servers. The major issue
with upgrading a server is that we must kill all running processes on
the server before performing the upgrade. Currently my upgrade process
is done by a single script run by the user which tries to upgrade all
servers at once. In order to run this script with the current batch
hacking system it's necessary to cancel all running batch hacking
jobs, upgrade the servers, then restart the batch hacking system.

I would like to make the upgrade process more automatic and more
seamless. Ultimately I want to create a new service that will
automatically manage buying and upgrading personal servers. This
service will need to be able to do it's work without stopping the
batch hacking system, and ideally it will do so with minimal
disruption. For instance, instead of forcibly killing all scripts on a
server it is about to upgrade, it would request the batch hacking
system vacate that server, then the task selector and memory manager
would respond by either killing low-value tasks or causing tasks to
start shifting their spawning away from the server to be upgraded.
