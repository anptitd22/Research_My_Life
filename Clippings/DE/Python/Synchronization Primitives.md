---
title: "Synchronization Primitives"
source: "https://docs.python.org/3.12/library/asyncio-sync.html"
author:
published:
created: 2026-04-13
description: "Source code: Lib/asyncio/locks.py asyncio synchronization primitives are designed to be similar to those of the threading module with two important caveats: asyncio primitives are not thread-safe, ..."
tags:
  - "clippings"
---
## Synchronization Primitives

**Source code:** [Lib/asyncio/locks.py](https://github.com/python/cpython/tree/3.12/Lib/asyncio/locks.py)

---

asyncio synchronization primitives are designed to be similar to those of the [`threading`](https://docs.python.org/3.12/library/threading.html#module-threading "threading: Thread-based parallelism.") module with two important caveats:

- asyncio primitives are not thread-safe, therefore they should not be used for OS thread synchronization (use [`threading`](https://docs.python.org/3.12/library/threading.html#module-threading "threading: Thread-based parallelism.") for that);
- methods of these synchronization primitives do not accept the *timeout* argument; use the [`asyncio.wait_for()`](https://docs.python.org/3.12/library/asyncio-task.html#asyncio.wait_for "asyncio.wait_for") function to perform operations with timeouts.

asyncio has the following basic synchronization primitives:

---

## Lock

*class* asyncio.Lock [¶](#asyncio.Lock "Link to this definition")

Implements a mutex lock for asyncio tasks. Not thread-safe.

An asyncio lock can be used to guarantee exclusive access to a shared resource.

The preferred way to use a Lock is an [`async with`](https://docs.python.org/3.12/reference/compound_stmts.html#async-with) statement:

```
lock = asyncio.Lock()

# ... later
async with lock:
    # access shared state
```

which is equivalent to:

```
lock = asyncio.Lock()

# ... later
await lock.acquire()
try:
    # access shared state
finally:
    lock.release()
```

Changed in version 3.10: Removed the *loop* parameter.

*async* acquire() [¶](#asyncio.Lock.acquire "Link to this definition")

Acquire the lock.

This method waits until the lock is *unlocked*, sets it to *locked* and returns `True`.

When more than one coroutine is blocked in waiting for the lock to be unlocked, only one coroutine eventually proceeds.

Acquiring a lock is *fair*: the coroutine that proceeds will be the first coroutine that started waiting on the lock.

release() [¶](#asyncio.Lock.release "Link to this definition")

Release the lock.

When the lock is *locked*, reset it to *unlocked* and return.

If the lock is *unlocked*, a [`RuntimeError`](https://docs.python.org/3.12/library/exceptions.html#RuntimeError "RuntimeError") is raised.

locked() [¶](#asyncio.Lock.locked "Link to this definition")

Return `True` if the lock is *locked*.

## Event

*class* asyncio.Event [¶](#asyncio.Event "Link to this definition")

An event object. Not thread-safe.

An asyncio event can be used to notify multiple asyncio tasks that some event has happened.

An Event object manages an internal flag that can be set to *true* with the method and reset to *false* with the method. The method blocks until the flag is set to *true*. The flag is set to *false* initially.

Changed in version 3.10: Removed the *loop* parameter.

Example:

```
async def waiter(event):
    print('waiting for it ...')
    await event.wait()
    print('... got it!')

async def main():
    # Create an Event object.
    event = asyncio.Event()

    # Spawn a Task to wait until 'event' is set.
    waiter_task = asyncio.create_task(waiter(event))

    # Sleep for 1 second and set the event.
    await asyncio.sleep(1)
    event.set()

    # Wait until the waiter task is finished.
    await waiter_task

asyncio.run(main())
```

*async* wait() [¶](#asyncio.Event.wait "Link to this definition")

Wait until the event is set.

If the event is set, return `True` immediately. Otherwise block until another task calls.

set() [¶](#asyncio.Event.set "Link to this definition")

Set the event.

All tasks waiting for event to be set will be immediately awakened.

clear() [¶](#asyncio.Event.clear "Link to this definition")

Clear (unset) the event.

Tasks awaiting on will now block until the method is called again.

is\_set() [¶](#asyncio.Event.is_set "Link to this definition")

Return `True` if the event is set.

## Condition

*class* asyncio.Condition(*lock=None*) [¶](#asyncio.Condition "Link to this definition")

A Condition object. Not thread-safe.

An asyncio condition primitive can be used by a task to wait for some event to happen and then get exclusive access to a shared resource.

In essence, a Condition object combines the functionality of an and a. It is possible to have multiple Condition objects share one Lock, which allows coordinating exclusive access to a shared resource between different tasks interested in particular states of that shared resource.

The optional *lock* argument must be a object or `None`. In the latter case a new Lock object is created automatically.

Changed in version 3.10: Removed the *loop* parameter.

The preferred way to use a Condition is an [`async with`](https://docs.python.org/3.12/reference/compound_stmts.html#async-with) statement:

```
cond = asyncio.Condition()

# ... later
async with cond:
    await cond.wait()
```

which is equivalent to:

```
cond = asyncio.Condition()

# ... later
await cond.acquire()
try:
    await cond.wait()
finally:
    cond.release()
```

*async* acquire() [¶](#asyncio.Condition.acquire "Link to this definition")

Acquire the underlying lock.

This method waits until the underlying lock is *unlocked*, sets it to *locked* and returns `True`.

notify(*n=1*) [¶](#asyncio.Condition.notify "Link to this definition")

Wake up at most *n* tasks (1 by default) waiting on this condition. The method is no-op if no tasks are waiting.

The lock must be acquired before this method is called and released shortly after. If called with an *unlocked* lock a [`RuntimeError`](https://docs.python.org/3.12/library/exceptions.html#RuntimeError "RuntimeError") error is raised.

locked() [¶](#asyncio.Condition.locked "Link to this definition")

Return `True` if the underlying lock is acquired.

notify\_all() [¶](#asyncio.Condition.notify_all "Link to this definition")

Wake up all tasks waiting on this condition.

This method acts like, but wakes up all waiting tasks.

The lock must be acquired before this method is called and released shortly after. If called with an *unlocked* lock a [`RuntimeError`](https://docs.python.org/3.12/library/exceptions.html#RuntimeError "RuntimeError") error is raised.

release() [¶](#asyncio.Condition.release "Link to this definition")

Release the underlying lock.

When invoked on an unlocked lock, a [`RuntimeError`](https://docs.python.org/3.12/library/exceptions.html#RuntimeError "RuntimeError") is raised.

*async* wait() [¶](#asyncio.Condition.wait "Link to this definition")

Wait until notified.

If the calling task has not acquired the lock when this method is called, a [`RuntimeError`](https://docs.python.org/3.12/library/exceptions.html#RuntimeError "RuntimeError") is raised.

This method releases the underlying lock, and then blocks until it is awakened by a or call. Once awakened, the Condition re-acquires its lock and this method returns `True`.

*async* wait\_for(*predicate*) [¶](#asyncio.Condition.wait_for "Link to this definition")

Wait until a predicate becomes *true*.

The predicate must be a callable which result will be interpreted as a boolean value. The method will repeatedly until the predicate evaluates to *true*. The final value is the return value.

## Semaphore

*class* asyncio.Semaphore(*value=1*) [¶](#asyncio.Semaphore "Link to this definition")

A Semaphore object. Not thread-safe.

A semaphore manages an internal counter which is decremented by each call and incremented by each call. The counter can never go below zero; when finds that it is zero, it blocks, waiting until some task calls.

The optional *value* argument gives the initial value for the internal counter (`1` by default). If the given value is less than `0` a [`ValueError`](https://docs.python.org/3.12/library/exceptions.html#ValueError "ValueError") is raised.

Changed in version 3.10: Removed the *loop* parameter.

The preferred way to use a Semaphore is an [`async with`](https://docs.python.org/3.12/reference/compound_stmts.html#async-with) statement:

```
sem = asyncio.Semaphore(10)

# ... later
async with sem:
    # work with shared resource
```

which is equivalent to:

```
sem = asyncio.Semaphore(10)

# ... later
await sem.acquire()
try:
    # work with shared resource
finally:
    sem.release()
```

*async* acquire() [¶](#asyncio.Semaphore.acquire "Link to this definition")

Acquire a semaphore.

If the internal counter is greater than zero, decrement it by one and return `True` immediately. If it is zero, wait until a is called and return `True`.

locked() [¶](#asyncio.Semaphore.locked "Link to this definition")

Returns `True` if semaphore can not be acquired immediately.

release() [¶](#asyncio.Semaphore.release "Link to this definition")

Release a semaphore, incrementing the internal counter by one. Can wake up a task waiting to acquire the semaphore.

Unlike, allows making more `release()` calls than `acquire()` calls.

## BoundedSemaphore

*class* asyncio.BoundedSemaphore(*value=1*) [¶](#asyncio.BoundedSemaphore "Link to this definition")

A bounded semaphore object. Not thread-safe.

Bounded Semaphore is a version of that raises a [`ValueError`](https://docs.python.org/3.12/library/exceptions.html#ValueError "ValueError") in if it increases the internal counter above the initial *value*.

Changed in version 3.10: Removed the *loop* parameter.

## Barrier

*class* asyncio.Barrier(*parties*) [¶](#asyncio.Barrier "Link to this definition")

A barrier object. Not thread-safe.

A barrier is a simple synchronization primitive that allows to block until *parties* number of tasks are waiting on it. Tasks can wait on the method and would be blocked until the specified number of tasks end up waiting on. At that point all of the waiting tasks would unblock simultaneously.

[`async with`](https://docs.python.org/3.12/reference/compound_stmts.html#async-with) can be used as an alternative to awaiting on.

The barrier can be reused any number of times.

Example:

```
async def example_barrier():
   # barrier with 3 parties
   b = asyncio.Barrier(3)

   # create 2 new waiting tasks
   asyncio.create_task(b.wait())
   asyncio.create_task(b.wait())

   await asyncio.sleep(0)
   print(b)

   # The third .wait() call passes the barrier
   await b.wait()
   print(b)
   print("barrier passed")

   await asyncio.sleep(0)
   print(b)

asyncio.run(example_barrier())
```

Result of this example is:

```
<asyncio.locks.Barrier object at 0x... [filling, waiters:2/3]>
<asyncio.locks.Barrier object at 0x... [draining, waiters:0/3]>
barrier passed
<asyncio.locks.Barrier object at 0x... [filling, waiters:0/3]>
```

Added in version 3.11.

*async* wait() [¶](#asyncio.Barrier.wait "Link to this definition")

Pass the barrier. When all the tasks party to the barrier have called this function, they are all unblocked simultaneously.

When a waiting or blocked task in the barrier is cancelled, this task exits the barrier which stays in the same state. If the state of the barrier is “filling”, the number of waiting task decreases by 1.

The return value is an integer in the range of 0 to `parties-1`, different for each task. This can be used to select a task to do some special housekeeping, e.g.:

```
...
async with barrier as position:
   if position == 0:
      # Only one task prints this
      print('End of *draining phase*')
```

This method may raise a exception if the barrier is broken or reset while a task is waiting. It could raise a [`CancelledError`](https://docs.python.org/3.12/library/asyncio-exceptions.html#asyncio.CancelledError "asyncio.CancelledError") if a task is cancelled.

*async* reset() [¶](#asyncio.Barrier.reset "Link to this definition")

Return the barrier to the default, empty state. Any tasks waiting on it will receive the exception.

If a barrier is broken it may be better to just leave it and create a new one.

*async* abort() [¶](#asyncio.Barrier.abort "Link to this definition")

Put the barrier into a broken state. This causes any active or future calls to to fail with the. Use this for example if one of the tasks needs to abort, to avoid infinite waiting tasks.

parties [¶](#asyncio.Barrier.parties "Link to this definition")

The number of tasks required to pass the barrier.

n\_waiting [¶](#asyncio.Barrier.n_waiting "Link to this definition")

The number of tasks currently waiting in the barrier while filling.

broken [¶](#asyncio.Barrier.broken "Link to this definition")

A boolean that is `True` if the barrier is in the broken state.

*exception* asyncio.BrokenBarrierError [¶](#asyncio.BrokenBarrierError "Link to this definition")

This exception, a subclass of [`RuntimeError`](https://docs.python.org/3.12/library/exceptions.html#RuntimeError "RuntimeError"), is raised when the object is reset or broken.

---

Changed in version 3.9: Acquiring a lock using `await lock` or `yield from lock` and/or [`with`](https://docs.python.org/3.12/reference/compound_stmts.html#with) statement (`with await lock`, `with (yield from lock)`) was removed. Use `async with lock` instead.