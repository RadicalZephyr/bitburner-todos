# How do Promises in Javascript really work?

How much can we actually do simultaneously? The usage of
`ns.nextWrite` to set a variable is pretty neat since we're just
registering a callback to run when the Promise is fulfilled.

But we can also create new promises and then... I don't really have
a good feel for what can be done with Promises, how they interact with
the runtime etc.

# Can I spawn WebWorkers inside Bitburner?

I don't really understand how WebWorkers function, and I don't know if
this would be useful but it potentially feels like absolutely cheating.
