---
title: "Thread states and the global interpreter lock"
source: "https://docs.python.org/3/c-api/threads.html"
author:
published:
created: 2026-04-11
description: "Unless on a free-threaded build of CPython, the Python interpreter is generally not thread-safe. In order to support multi-threaded Python programs, there’s a global lock, called the global interpr..."
tags:
  - "clippings"
---
## Thread states and the global interpreter lock

Unless on a [free-threaded build](https://docs.python.org/3/glossary.html#term-free-threaded-build) of [CPython](https://docs.python.org/3/glossary.html#term-CPython), the Python interpreter is generally not thread-safe. In order to support multi-threaded Python programs, there’s a global lock, called the [global interpreter lock](https://docs.python.org/3/glossary.html#term-global-interpreter-lock) or [GIL](https://docs.python.org/3/glossary.html#term-GIL), that must be held by a thread before accessing Python objects. Without the lock, even the simplest operations could cause problems in a multi-threaded program: for example, when two threads simultaneously increment the reference count of the same object, the reference count could end up being incremented only once instead of twice.

As such, only a thread that holds the GIL may operate on Python objects or invoke Python’s C API.

In order to emulate concurrency, the interpreter regularly tries to switch threads between bytecode instructions (see [`sys.setswitchinterval()`](https://docs.python.org/3/library/sys.html#sys.setswitchinterval "sys.setswitchinterval")). This is why locks are also necessary for thread-safety in pure-Python code.

Additionally, the global interpreter lock is released around blocking I/O operations, such as reading or writing to a file. From the C API, this is done by.

The Python interpreter keeps some thread-local information inside a data structure called, known as a [thread state](https://docs.python.org/3/glossary.html#term-thread-state). Each thread has a thread-local pointer to a `PyThreadState`; a thread state referenced by this pointer is considered to be [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state).

A thread can only have one [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) at a time. An attached thread state is typically analogous with holding the GIL, except on free-threaded builds. On builds with the GIL enabled, attaching a thread state will block until the GIL can be acquired. However, even on builds with the GIL disabled, it is still required to have an attached thread state, as the interpreter needs to keep track of which threads may access Python objects.

Note

Even on the free-threaded build, attaching a thread state may block, as the GIL can be re-enabled or threads might be temporarily suspended (such as during a garbage collection).

Generally, there will always be an attached thread state when using Python’s C API, including during embedding and when implementing methods, so it’s uncommon to need to set up a thread state on your own. Only in some specific cases, such as in a block or in a fresh thread, will the thread not have an attached thread state. If uncertain, check if returns `NULL`.

If it turns out that you do need to create a thread state, call followed by, or use the dangerous function.

## Detaching the thread state from extension code

Most extension code manipulating the [thread state](https://docs.python.org/3/glossary.html#term-thread-state) has the following simple structure:

```c
Save the thread state in a local variable.
... Do some blocking I/O operation ...
Restore the thread state from the local variable.
```

This is so common that a pair of macros exists to simplify it:

```c
Py_BEGIN_ALLOW_THREADS
... Do some blocking I/O operation ...
Py_END_ALLOW_THREADS
```

The macro opens a new block and declares a hidden local variable; the macro closes the block.

The block above expands to the following code:

```c
PyThreadState *_save;

_save = PyEval_SaveThread();
... Do some blocking I/O operation ...
PyEval_RestoreThread(_save);
```

Here is how these functions work:

The attached thread state implies that the GIL is held for the interpreter. To detach it, is called and the result is stored in a local variable.

By detaching the thread state, the GIL is released, which allows other threads to attach to the interpreter and execute while the current thread performs blocking I/O. When the I/O operation is complete, the old thread state is reattached by calling, which will wait until the GIL can be acquired.

Note

Performing blocking I/O is the most common use case for detaching the thread state, but it is also useful to call it over long-running native code that doesn’t need access to Python objects or Python’s C API. For example, the standard [`zlib`](https://docs.python.org/3/library/zlib.html#module-zlib "zlib: Low-level interface to compression and decompression routines compatible with gzip.") and [`hashlib`](https://docs.python.org/3/library/hashlib.html#module-hashlib "hashlib: Secure hash and message digest algorithms.") modules detach the [thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) when compressing or hashing data.

On a [free-threaded build](https://docs.python.org/3/glossary.html#term-free-threaded-build), the [GIL](https://docs.python.org/3/glossary.html#term-GIL) is usually out of the question, but **detaching the thread state is still required**, because the interpreter periodically needs to block all threads to get a consistent view of Python objects without the risk of race conditions. For example, CPython currently suspends all threads for a short period of time while running the garbage collector.

Warning

Detaching the thread state can lead to unexpected behavior during interpreter finalization. See [Cautions regarding runtime finalization](https://docs.python.org/3/c-api/interp-lifecycle.html#cautions-regarding-runtime-finalization) for more details.

### APIs

The following macros are normally used without a trailing semicolon; look for example usage in the Python source distribution.

Note

These macros are still necessary on the [free-threaded build](https://docs.python.org/3/glossary.html#term-free-threaded-build) to prevent deadlocks.

Py\_BEGIN\_ALLOW\_THREADS [¶](#c.Py_BEGIN_ALLOW_THREADS "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

This macro expands to `{ PyThreadState *_save; _save = PyEval_SaveThread();`. Note that it contains an opening brace; it must be matched with a following macro. See above for further discussion of this macro.

Py\_END\_ALLOW\_THREADS [¶](#c.Py_END_ALLOW_THREADS "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

This macro expands to `PyEval_RestoreThread(_save); }`. Note that it contains a closing brace; it must be matched with an earlier macro. See above for further discussion of this macro.

Py\_BLOCK\_THREADS [¶](#c.Py_BLOCK_THREADS "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

This macro expands to `PyEval_RestoreThread(_save);`: it is equivalent to without the closing brace.

Py\_UNBLOCK\_THREADS [¶](#c.Py_UNBLOCK_THREADS "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

This macro expands to `_save = PyEval_SaveThread();`: it is equivalent to without the opening brace and variable declaration.

## Non-Python created threads

When threads are created using the dedicated Python APIs (such as the [`threading`](https://docs.python.org/3/library/threading.html#module-threading "threading: Thread-based parallelism.") module), a thread state is automatically associated with them, However, when a thread is created from native code (for example, by a third-party library with its own thread management), it doesn’t hold an attached thread state.

If you need to call Python code from these threads (often this will be part of a callback API provided by the aforementioned third-party library), you must first register these threads with the interpreter by creating a new thread state and attaching it.

The most robust way to do this is through followed by.

Note

`PyThreadState_New` requires an argument pointing to the desired interpreter; such a pointer can be acquired via a call to [`PyInterpreterState_Get()`](https://docs.python.org/3/c-api/subinterpreters.html#c.PyInterpreterState_Get "PyInterpreterState_Get") from the code where the thread was created.

For example:

```c
/* The return value of PyInterpreterState_Get() from the
   function that created this thread. */
PyInterpreterState *interp = thread_data->interp;

/* Create a new thread state for the interpreter. It does not start out
   attached. */
PyThreadState *tstate = PyThreadState_New(interp);

/* Attach the thread state, which will acquire the GIL. */
PyThreadState_Swap(tstate);

/* Perform Python actions here. */
result = CallSomeFunction();
/* evaluate result or handle exception */

/* Destroy the thread state. No Python API allowed beyond this point. */
PyThreadState_Clear(tstate);
PyThreadState_DeleteCurrent();
```

Warning

If the interpreter finalized before `PyThreadState_Swap` was called, then `interp` will be a dangling pointer!

## Legacy API

Another common pattern to call Python code from a non-Python thread is to use followed by a call to.

These functions do not work well when multiple interpreters exist in the Python process. If no Python interpreter has ever been used in the current thread (which is common for threads created outside Python), `PyGILState_Ensure` will create and attach a thread state for the “main” interpreter (the first interpreter in the Python process).

Additionally, these functions have thread-safety issues during interpreter finalization. Using `PyGILState_Ensure` during finalization will likely crash the process.

Usage of these functions look like such:

```c
PyGILState_STATE gstate;
gstate = PyGILState_Ensure();

/* Perform Python actions here. */
result = CallSomeFunction();
/* evaluate result or handle exception */

/* Release the thread. No Python API allowed beyond this point. */
PyGILState_Release(gstate);
```

## Cautions about fork()

Another important thing to note about threads is their behaviour in the face of the C `fork()` call. On most systems with `fork()`, after a process forks only the thread that issued the fork will exist. This has a concrete impact both on how locks must be handled and on all stored state in CPython’s runtime.

The fact that only the “current” thread remains means any locks held by other threads will never be released. Python solves this for [`os.fork()`](https://docs.python.org/3/library/os.html#os.fork "os.fork") by acquiring the locks it uses internally before the fork, and releasing them afterwards. In addition, it resets any [Lock objects](https://docs.python.org/3/library/threading.html#lock-objects) in the child. When extending or embedding Python, there is no way to inform Python of additional (non-Python) locks that need to be acquired before or reset after a fork. OS facilities such as `pthread_atfork()` would need to be used to accomplish the same thing. Additionally, when extending or embedding Python, calling `fork()` directly rather than through `os.fork()` (and returning to or calling into Python) may result in a deadlock by one of Python’s internal locks being held by a thread that is defunct after the fork. [`PyOS_AfterFork_Child()`](https://docs.python.org/3/c-api/sys.html#c.PyOS_AfterFork_Child "PyOS_AfterFork_Child") tries to reset the necessary locks, but is not always able to.

The fact that all other threads go away also means that CPython’s runtime state there must be cleaned up properly, which [`os.fork()`](https://docs.python.org/3/library/os.html#os.fork "os.fork") does. This means finalizing all other objects belonging to the current interpreter and all other [`PyInterpreterState`](https://docs.python.org/3/c-api/subinterpreters.html#c.PyInterpreterState "PyInterpreterState") objects. Due to this and the special nature of the [“main” interpreter](https://docs.python.org/3/c-api/subinterpreters.html#sub-interpreter-support), `fork()` should only be called in that interpreter’s “main” thread, where the CPython global runtime was originally initialized. The only exception is if `exec()` will be called immediately after.

## High-level APIs

These are the most commonly used types and functions when writing multi-threaded C extensions.

type PyThreadState [¶](#c.PyThreadState "Link to this definition")  

*Part of the [Limited API](https://docs.python.org/3/c-api/stable.html#stable) (as an opaque struct).*

This data structure represents the state of a single thread. The only public data member is:

[PyInterpreterState](https://docs.python.org/3/c-api/subinterpreters.html#c.PyInterpreterState "PyInterpreterState") \*interp [¶](#c.PyThreadState.interp "Link to this definition")  

This thread’s interpreter state.

void PyEval\_InitThreads() [¶](#c.PyEval_InitThreads "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Deprecated function which does nothing.

In Python 3.6 and older, this function created the GIL if it didn’t exist.

Changed in version 3.9: The function now does nothing.

Changed in version 3.7: This function is now called by [`Py_Initialize()`](https://docs.python.org/3/c-api/interp-lifecycle.html#c.Py_Initialize "Py_Initialize"), so you don’t have to call it yourself anymore.

Changed in version 3.2: This function cannot be called before [`Py_Initialize()`](https://docs.python.org/3/c-api/interp-lifecycle.html#c.Py_Initialize "Py_Initialize") anymore.

Deprecated since version 3.9.

\*PyEval\_SaveThread() [¶](#c.PyEval_SaveThread "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Detach the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) and return it. The thread will have no [thread state](https://docs.python.org/3/glossary.html#term-thread-state) upon returning.

void PyEval\_RestoreThread( \*tstate) [¶](#c.PyEval_RestoreThread "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Set the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) to *tstate*. The passed [thread state](https://docs.python.org/3/glossary.html#term-thread-state) **should not** be attached, otherwise deadlock ensues. *tstate* will be attached upon returning.

Note

Calling this function from a thread when the runtime is finalizing will hang the thread until the program exits, even if the thread was not created by Python. Refer to [Cautions regarding runtime finalization](https://docs.python.org/3/c-api/interp-lifecycle.html#cautions-regarding-runtime-finalization) for more details.

Changed in version 3.14: Hangs the current thread, rather than terminating it, if called while the interpreter is finalizing.

\*PyThreadState\_Get() [¶](#c.PyThreadState_Get "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Return the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state). If the thread has no attached thread state, (such as when inside of block), then this issues a fatal error (so that the caller needn’t check for `NULL`).

See also.

\*PyThreadState\_GetUnchecked() [¶](#c.PyThreadState_GetUnchecked "Link to this definition")  

Similar to, but don’t kill the process with a fatal error if it is NULL. The caller is responsible to check if the result is NULL.

Added in version 3.13: In Python 3.5 to 3.12, the function was private and known as `_PyThreadState_UncheckedGet()`.

\*PyThreadState\_Swap( \*tstate) [¶](#c.PyThreadState_Swap "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Set the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) to *tstate*, and return the [thread state](https://docs.python.org/3/glossary.html#term-thread-state) that was attached prior to calling.

This function is safe to call without an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state); it will simply return `NULL` indicating that there was no prior thread state.

See also

Note

Similar to, this function will hang the thread if the runtime is finalizing.

## GIL-state APIs

The following functions use thread-local storage, and are not compatible with sub-interpreters:

type PyGILState\_STATE [¶](#c.PyGILState_STATE "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

The type of the value returned by and passed to.

enumerator PyGILState\_LOCKED [¶](#c.PyGILState_STATE.PyGILState_LOCKED "Link to this definition")  

The GIL was already held when was called.

enumerator PyGILState\_UNLOCKED [¶](#c.PyGILState_STATE.PyGILState_UNLOCKED "Link to this definition")  

The GIL was not held when was called.

PyGILState\_Ensure() [¶](#c.PyGILState_Ensure "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Ensure that the current thread is ready to call the Python C API regardless of the current state of Python, or of the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state). This may be called as many times as desired by a thread as long as each call is matched with a call to. In general, other thread-related APIs may be used between and `PyGILState_Release()` calls as long as the thread state is restored to its previous state before the Release(). For example, normal usage of the and macros is acceptable.

The return value is an opaque “handle” to the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) when was called, and must be passed to to ensure Python is left in the same state. Even though recursive calls are allowed, these handles *cannot* be shared - each unique call to `PyGILState_Ensure()` must save the handle for its call to `PyGILState_Release()`.

When the function returns, there will be an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) and the thread will be able to call arbitrary Python code. Failure is a fatal error.

Warning

Calling this function when the runtime is finalizing is unsafe. Doing so will either hang the thread until the program ends, or fully crash the interpreter in rare cases. Refer to [Cautions regarding runtime finalization](https://docs.python.org/3/c-api/interp-lifecycle.html#cautions-regarding-runtime-finalization) for more details.

Changed in version 3.14: Hangs the current thread, rather than terminating it, if called while the interpreter is finalizing.

void PyGILState\_Release() [¶](#c.PyGILState_Release "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Release any resources previously acquired. After this call, Python’s state will be the same as it was prior to the corresponding call (but generally this state will be unknown to the caller, hence the use of the GILState API).

Every call to must be matched by a call to on the same thread.

\*PyGILState\_GetThisThreadState() [¶](#c.PyGILState_GetThisThreadState "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Get the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) for this thread. May return `NULL` if no GILState API has been used on the current thread. Note that the main thread always has such a thread-state, even if no auto-thread-state call has been made on the main thread. This is mainly a helper/diagnostic function.

Note

This function may return non- `NULL` even when the [thread state](https://docs.python.org/3/glossary.html#term-thread-state) is detached. Prefer or for most cases.

See also

int PyGILState\_Check() [¶](#c.PyGILState_Check "Link to this definition")  

Return `1` if the current thread is holding the [GIL](https://docs.python.org/3/glossary.html#term-GIL) and `0` otherwise. This function can be called from any thread at any time. Only if it has had its [thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) initialized via will it return `1`. This is mainly a helper/diagnostic function. It can be useful for example in callback contexts or memory allocation functions when knowing that the GIL is locked can allow the caller to perform sensitive actions or otherwise behave differently.

Note

If the current Python process has ever created a subinterpreter, this function will *always* return `1`. Prefer for most cases.

Added in version 3.4.

## Low-level APIs

\*PyThreadState\_New([PyInterpreterState](https://docs.python.org/3/c-api/subinterpreters.html#c.PyInterpreterState "PyInterpreterState") \*interp) [¶](#c.PyThreadState_New "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Create a new thread state object belonging to the given interpreter object. An [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) is not needed.

void PyThreadState\_Clear( \*tstate) [¶](#c.PyThreadState_Clear "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Reset all information in a [thread state](https://docs.python.org/3/glossary.html#term-thread-state) object. *tstate* must be [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state)

Changed in version 3.9: This function now calls the `PyThreadState.on_delete` callback. Previously, that happened in.

Changed in version 3.13: The `PyThreadState.on_delete` callback was removed.

void PyThreadState\_Delete( \*tstate) [¶](#c.PyThreadState_Delete "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Destroy a [thread state](https://docs.python.org/3/glossary.html#term-thread-state) object. *tstate* should not be [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state) to any thread. *tstate* must have been reset with a previous call to.

void PyThreadState\_DeleteCurrent(void) [¶](#c.PyThreadState_DeleteCurrent "Link to this definition")  

Detach the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) (which must have been reset with a previous call to ) and then destroy it.

No [thread state](https://docs.python.org/3/glossary.html#term-thread-state) will be [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state) upon returning.

[PyFrameObject](https://docs.python.org/3/c-api/frame.html#c.PyFrameObject "PyFrameObject") \*PyThreadState\_GetFrame( \*tstate) [¶](#c.PyThreadState_GetFrame "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable) since version 3.10.*

Get the current frame of the Python thread state *tstate*.

Return a [strong reference](https://docs.python.org/3/glossary.html#term-strong-reference). Return `NULL` if no frame is currently executing.

See also [`PyEval_GetFrame()`](https://docs.python.org/3/c-api/reflection.html#c.PyEval_GetFrame "PyEval_GetFrame").

*tstate* must not be `NULL`, and must be [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state).

Added in version 3.9.

uint64\_t PyThreadState\_GetID( \*tstate) [¶](#c.PyThreadState_GetID "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable) since version 3.10.*

Get the unique [thread state](https://docs.python.org/3/glossary.html#term-thread-state) identifier of the Python thread state *tstate*.

*tstate* must not be `NULL`, and must be [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state).

Added in version 3.9.

[PyInterpreterState](https://docs.python.org/3/c-api/subinterpreters.html#c.PyInterpreterState "PyInterpreterState") \*PyThreadState\_GetInterpreter( \*tstate) [¶](#c.PyThreadState_GetInterpreter "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable) since version 3.10.*

Get the interpreter of the Python thread state *tstate*.

*tstate* must not be `NULL`, and must be [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state).

Added in version 3.9.

void PyThreadState\_EnterTracing( \*tstate) [¶](#c.PyThreadState_EnterTracing "Link to this definition")  

Suspend tracing and profiling in the Python thread state *tstate*.

Resume them using the function.

Added in version 3.11.

void PyThreadState\_LeaveTracing( \*tstate) [¶](#c.PyThreadState_LeaveTracing "Link to this definition")  

Resume tracing and profiling in the Python thread state *tstate* suspended by the function.

See also [`PyEval_SetTrace()`](https://docs.python.org/3/c-api/profiling.html#c.PyEval_SetTrace "PyEval_SetTrace") and [`PyEval_SetProfile()`](https://docs.python.org/3/c-api/profiling.html#c.PyEval_SetProfile "PyEval_SetProfile") functions.

Added in version 3.11.

int PyUnstable\_ThreadState\_SetStackProtection( \*tstate, void \*stack\_start\_addr, size\_t stack\_size) [¶](#c.PyUnstable_ThreadState_SetStackProtection "Link to this definition")  

*This is [Unstable API](https://docs.python.org/3/c-api/stable.html#unstable-c-api). It may change without warning in minor releases.*

Set the stack protection start address and stack protection size of a Python thread state.

On success, return `0`. On failure, set an exception and return `-1`.

CPython implements [recursion control](https://docs.python.org/3/c-api/exceptions.html#recursion) for C code by raising [`RecursionError`](https://docs.python.org/3/library/exceptions.html#RecursionError "RecursionError") when it notices that the machine execution stack is close to overflow. See for example the [`Py_EnterRecursiveCall()`](https://docs.python.org/3/c-api/exceptions.html#c.Py_EnterRecursiveCall "Py_EnterRecursiveCall") function. For this, it needs to know the location of the current thread’s stack, which it normally gets from the operating system. When the stack is changed, for example using context switching techniques like the Boost library’s `boost::context`, you must call to inform CPython of the change.

Call either before or after changing the stack. Do not call any other Python C API between the call and the stack change.

See for undoing this operation.

Added in version 3.15.

void PyUnstable\_ThreadState\_ResetStackProtection( \*tstate) [¶](#c.PyUnstable_ThreadState_ResetStackProtection "Link to this definition")  

*This is [Unstable API](https://docs.python.org/3/c-api/stable.html#unstable-c-api). It may change without warning in minor releases.*

Reset the stack protection start address and stack protection size of a Python thread state to the operating system defaults.

See for an explanation.

Added in version 3.15.

[PyObject](https://docs.python.org/3/c-api/structures.html#c.PyObject "PyObject") \*PyThreadState\_GetDict() [¶](#c.PyThreadState_GetDict "Link to this definition")  

*Return value: Borrowed reference.* *Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Return a dictionary in which extensions can store thread-specific state information. Each extension should use a unique key to use to store state in the dictionary. It is okay to call this function when no [thread state](https://docs.python.org/3/glossary.html#term-thread-state) is [attached](https://docs.python.org/3/glossary.html#term-attached-thread-state). If this function returns `NULL`, no exception has been raised and the caller should assume no thread state is attached.

void PyEval\_AcquireThread( \*tstate) [¶](#c.PyEval_AcquireThread "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

[Attach](https://docs.python.org/3/glossary.html#term-attached-thread-state) *tstate* to the current thread, which must not be `NULL` or already attached.

The calling thread must not already have an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).

Note

Calling this function from a thread when the runtime is finalizing will hang the thread until the program exits, even if the thread was not created by Python. Refer to [Cautions regarding runtime finalization](https://docs.python.org/3/c-api/interp-lifecycle.html#cautions-regarding-runtime-finalization) for more details.

Changed in version 3.8: Updated to be consistent with,, and, and terminate the current thread if called while the interpreter is finalizing.

Changed in version 3.14: Hangs the current thread, rather than terminating it, if called while the interpreter is finalizing.

is a higher-level function which is always available (even when threads have not been initialized).

void PyEval\_ReleaseThread( \*tstate) [¶](#c.PyEval_ReleaseThread "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Detach the [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state). The *tstate* argument, which must not be `NULL`, is only used to check that it represents the attached thread state — if it isn’t, a fatal error is reported.

is a higher-level function which is always available (even when threads have not been initialized).

## Asynchronous notifications

A mechanism is provided to make asynchronous notifications to the main interpreter thread. These notifications take the form of a function pointer and a void pointer argument.

int Py\_AddPendingCall(int (\*func)(void\*), void \*arg) [¶](#c.Py_AddPendingCall "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Schedule a function to be called from the main interpreter thread. On success, `0` is returned and *func* is queued for being called in the main thread. On failure, `-1` is returned without setting any exception.

When successfully queued, *func* will be *eventually* called from the main interpreter thread with the argument *arg*. It will be called asynchronously with respect to normally running Python code, but with both these conditions met:

- on a [bytecode](https://docs.python.org/3/glossary.html#term-bytecode) boundary;
- with the main thread holding an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) (*func* can therefore use the full C API).

*func* must return `0` on success, or `-1` on failure with an exception set. *func* won’t be interrupted to perform another asynchronous notification recursively, but it can still be interrupted to switch threads if the [thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state) is detached.

This function doesn’t need an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state). However, to call this function in a subinterpreter, the caller must have an attached thread state. Otherwise, the function *func* can be scheduled to be called from the wrong interpreter.

Warning

This is a low-level function, only useful for very special cases. There is no guarantee that *func* will be called as quick as possible. If the main thread is busy executing a system call, *func* won’t be called before the system call returns. This function is generally **not** suitable for calling Python code from arbitrary C threads. Instead, use the.

Added in version 3.1.

Changed in version 3.9: If this function is called in a subinterpreter, the function *func* is now scheduled to be called from the subinterpreter, rather than being called from the main interpreter. Each subinterpreter now has its own list of scheduled calls.

Changed in version 3.12: This function now always schedules *func* to be run in the main interpreter.

int Py\_MakePendingCalls(void) [¶](#c.Py_MakePendingCalls "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Execute all pending calls. This is usually executed automatically by the interpreter.

This function returns `0` on success, and returns `-1` with an exception set on failure.

If this is not called in the main thread of the main interpreter, this function does nothing and returns `0`. The caller must hold an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).

Added in version 3.1.

Changed in version 3.12: This function only runs pending calls in the main interpreter.

int PyThreadState\_SetAsyncExc(unsigned long id, [PyObject](https://docs.python.org/3/c-api/structures.html#c.PyObject "PyObject") \*exc) [¶](#c.PyThreadState_SetAsyncExc "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Asynchronously raise an exception in a thread. The *id* argument is the thread id of the target thread; *exc* is the exception object to be raised. This function does not steal any references to *exc*. To prevent naive misuse, you must write your own C extension to call this. Must be called with an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state). Returns the number of thread states modified; this is normally one, but will be zero if the thread id isn’t found. If *exc* is `NULL`, the pending exception (if any) for the thread is cleared. This raises no exceptions.

Changed in version 3.7: The type of the *id* parameter changed from long to unsigned long.

## Operating system thread APIs

PYTHREAD\_INVALID\_THREAD\_ID [¶](#c.PYTHREAD_INVALID_THREAD_ID "Link to this definition")  

Sentinel value for an invalid thread ID.

This is currently equivalent to `(unsigned long)-1`.

unsigned long PyThread\_start\_new\_thread(void (\*func)(void\*), void \*arg) [¶](#c.PyThread_start_new_thread "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Start function *func* in a new thread with argument *arg*. The resulting thread is not intended to be joined.

*func* must not be `NULL`, but *arg* may be `NULL`.

On success, this function returns the identifier of the new thread; on failure, this returns.

The caller does not need to hold an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).

unsigned long PyThread\_get\_thread\_ident(void) [¶](#c.PyThread_get_thread_ident "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Return the identifier of the current thread, which will never be zero.

This function cannot fail, and the caller does not need to hold an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).

See also

[`threading.get_ident()`](https://docs.python.org/3/library/threading.html#threading.get_ident "threading.get_ident")

[PyObject](https://docs.python.org/3/c-api/structures.html#c.PyObject "PyObject") \*PyThread\_GetInfo(void) [¶](#c.PyThread_GetInfo "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable) since version 3.3.*

Get general information about the current thread in the form of a [struct sequence](https://docs.python.org/3/c-api/tuple.html#struct-sequence-objects) object. This information is accessible as [`sys.thread_info`](https://docs.python.org/3/library/sys.html#sys.thread_info "sys.thread_info") in Python.

On success, this returns a new [strong reference](https://docs.python.org/3/glossary.html#term-strong-reference) to the thread information; on failure, this returns `NULL` with an exception set.

The caller must hold an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).

PY\_HAVE\_THREAD\_NATIVE\_ID [¶](#c.PY_HAVE_THREAD_NATIVE_ID "Link to this definition")  

This macro is defined when the system supports native thread IDs.

unsigned long PyThread\_get\_thread\_native\_id(void) [¶](#c.PyThread_get_thread_native_id "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable) on platforms with native thread IDs.*

Get the native identifier of the current thread as it was assigned by the operating system’s kernel, which will never be less than zero.

This function is only available when is defined.

This function cannot fail, and the caller does not need to hold an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).

See also

[`threading.get_native_id()`](https://docs.python.org/3/library/threading.html#threading.get_native_id "threading.get_native_id")

void PyThread\_exit\_thread(void) [¶](#c.PyThread_exit_thread "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Terminate the current thread. This function is generally considered unsafe and should be avoided. It is kept solely for backwards compatibility.

This function is only safe to call if all functions in the full call stack are written to safely allow it.

Warning

If the current system uses POSIX threads (also known as “pthreads”), this calls *[pthread\_exit(3)](https://manpages.debian.org/pthread_exit\(3\))*, which attempts to unwind the stack and call C++ destructors on some libc implementations. However, if a `noexcept` function is reached, it may terminate the process. Other systems, such as macOS, do unwinding.

On Windows, this function calls `_endthreadex()`, which kills the thread without calling C++ destructors.

In any case, there is a risk of corruption on the thread’s stack.

Deprecated since version 3.14.

void PyThread\_init\_thread(void) [¶](#c.PyThread_init_thread "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Initialize `PyThread*` APIs. Python executes this function automatically, so there’s little need to call it from an extension module.

int PyThread\_set\_stacksize(size\_t size) [¶](#c.PyThread_set_stacksize "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Set the stack size of the current thread to *size* bytes.

This function returns `0` on success, `-1` if *size* is invalid, or `-2` if the system does not support changing the stack size. This function does not set exceptions.

The caller does not need to hold an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).

size\_t PyThread\_get\_stacksize(void) [¶](#c.PyThread_get_stacksize "Link to this definition")  

*Part of the [Stable ABI](https://docs.python.org/3/c-api/stable.html#stable).*

Return the stack size of the current thread in bytes, or `0` if the system’s default stack size is in use.

The caller does not need to hold an [attached thread state](https://docs.python.org/3/glossary.html#term-attached-thread-state).