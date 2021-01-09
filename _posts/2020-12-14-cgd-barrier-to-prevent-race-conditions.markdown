---
layout: post
title: "CGD Barrier to prevent race conditions"
date: 2020-12-14 20:40:20 GMT
categories: swift gcd memory management
---

We all been there:

```
Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'The number of rows contained in an existing section after the update must be equal to the number of rows contained in that section before the update, plus or minus the number of rows inserted or deleted from that section and plus or minus the number of rows moved into or out of that section (0 moved in, 0 moved out).'
```

The table view data is backed in an ivar (an array), which is asynchronously updated in a separate thread. But swift objects are not thread safe, writing to an instance variable is not atomic. Two threads can write at the same time, and that might cause memory corruption, or in a more benign case, to break the synchronization with other threads. 

The ideal situation to avoid this behavior is that reads happen *synchronously* and *concurrently*, whereas writes can be *asynchronous* and must be **the only thing happening to the reference**. 

That's exactly what GCD introduces when marking an execution block with the `.barrier` flag. When dispatching a block with such a flag, the queue will wait for all the running blocks to finish before starting the one marked, and if it receives any new block, it will wait until finishing the barrier block before starting with them.

First, we need to define a private queue to mediate access to the ivar

```
private let accessQueue = DispatchQueue(label: "me.volonbolon.isolation.queue",
                                            attributes: .concurrent)
``` 

Then, we can use it safely read from our ivar 

```
var books: [Book] {
        get {
            accessQueue.sync {
                return self._books
            }
        }
    }
```

We wait for the queue to give us the ivar. Because the access queue is concurrent (private queues are serial by default, but we instantiate it as `.concurrent`), many reads can be performed simultaneously. 

On the other hand, when writing, we want to block the thread, and thus, we dispatch the execution block asynchronously to free the caller quickly, and we mark it with the `.barrier` flag to make sure it will be run alone. 

```
        self.accessQueue.async(group: nil,
                               qos: .userInitiated,
                               flags: .barrier) {
            // update books
            // And the reload appropriate cells
            DispatchQueue.main.async {
                self.tableView.performBatchUpdates({
                    self.tableView.insertRows(at: indices, with: .automatic)
                },
                completion: nil)
            }
        }
```