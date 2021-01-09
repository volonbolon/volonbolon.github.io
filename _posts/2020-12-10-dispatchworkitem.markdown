---
layout: post
title: "DispatchWorkItem"
date: 2020-12-10 19:02:56 GMT
tags: swift gcd iOS
---

Dispatch Queues are great, but sometimes, the *Fire-and-forget* approach is not quite right. We do have [NSOperation](https://developer.apple.com/documentation/foundation/nsoperation), to encapsulate complex tasks, but what if we are dealing with something simple, something we can handle with Dispatch Queues, and yet, we need to cancel or create a simple dependency hierarchy? In that case, [DispatchWorkItem](https://developer.apple.com/documentation/dispatch/dispatchworkitem) might save the day. 

A DispatchWorkItem encapsulates work to be performed on a dispatch queue or within a dispatch group. You can also use a work item as a [DispatchSource](https://developer.apple.com/documentation/dispatch/dispatchsource) event, registration, or cancellation handler.

### Cancellation 
The canonical example of DispatchWorkItem is retrieval of suggestions for a search bar. We have a search bar, and the user starts typing. We will wait for the user to stop typing, at least for a while, to query the backend. 

```
class SearchController {
    var pendingWorkItem: DispatchWorkItem?
    
    func getSearchResults(query: String) {
        if !(pendingWorkItem?.isCancelled ?? false) {
            pendingWorkItem?.cancel()
        }
        let pending = DispatchWorkItem(qos: .userInitiated, flags: .assignCurrentContext) {
            print("Querying for '\(query)'")
        }
        pendingWorkItem = pending
        DispatchQueue.global().asyncAfter(deadline: .now() + .milliseconds(30), execute: pending)
    }
}

let search = SearchController()
search.getSearchResults(query: "q")
search.getSearchResults(query: "qw")
sleep(UInt32(Int.random(in: 1...3)))
search.getSearchResults(query: "qwe")

// Querying for 'qw'
// Querying for 'qwe'

```

Here we ignore the first query because, even before activating the work item, it got cancelled. 

### Dependency
We can also build a simple dependency structure. Let say we need two task, one after the other has been finished. Perhaps, we want to signal in the UI the end of a large computation. In that case, we can link work items with `notify` 

```
let userQueue = DispatchQueue.global(qos: .userInitiated)

func task1() {
  print("Task 1 started")
  sleep(2)
  print("Task 1 finished")
}
func task2() {
  print("Updating UI")
}

let backgroundWorkItem = DispatchWorkItem {
  task1()
}

let updateUIWorkItem = DispatchWorkItem {
  task2()
}

backgroundWorkItem.notify(queue: userQueue,
                          execute: updateUIWorkItem)

userQueue.async(execute: backgroundWorkItem)

// Task 1 started
// Task 1 finished
// Updating UI
```
 
We can also block the thread until the workItem is done calling `wait`. In that case the system, if necessary/possible, will increase the priority of other tasks in its queue.

```
let userQueue = DispatchQueue.global(qos: .userInitiated)

func task1() {
  print("Task 1 started")
  sleep(2)
  print("Task 1 finished")
}
func task2() {
  print("Updating UI")
}

let backgroundWorkItem = DispatchWorkItem {
  task1()
}

userQueue.async(execute: backgroundWorkItem)

backgroundWorkItem.wait()

print("We've passed the work item")

// Task 1 started
// Task 1 finished
// We've passed the work item

```
