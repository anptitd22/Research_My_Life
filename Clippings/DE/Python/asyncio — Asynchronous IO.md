---
title: "asyncio — Asynchronous I/O"
source: "https://docs.python.org/3/library/asyncio.html"
author:
published:
created: 2026-04-12
description: "Hello World!: asyncio is a library to write concurrent code using the async/await syntax. asyncio is used as a foundation for multiple Python asynchronous frameworks that provide high-performance n..."
tags:
  - "clippings"
---
## asyncio — Asynchronous I/O

---

asyncio is a library to write **concurrent** code using the **async/await** syntax.

asyncio is used as a foundation for multiple Python asynchronous frameworks that provide high-performance network and web-servers, database connection libraries, distributed task queues, etc.

asyncio is often a perfect fit for IO-bound and high-level **structured** network code.

See also

[A Conceptual Overview of asyncio](https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html#a-conceptual-overview-of-asyncio)

Explanation of the fundamentals of asyncio.

asyncio provides a set of **high-level** APIs to:

- [run Python coroutines](https://docs.python.org/3/library/asyncio-task.html#coroutine) concurrently and have full control over their execution;
- perform [network IO and IPC](https://docs.python.org/3/library/asyncio-stream.html#asyncio-streams);
- control [subprocesses](https://docs.python.org/3/library/asyncio-subprocess.html#asyncio-subprocess);
- distribute tasks via [queues](https://docs.python.org/3/library/asyncio-queue.html#asyncio-queues);
- [synchronize](https://docs.python.org/3/library/asyncio-sync.html#asyncio-sync) concurrent code;

Additionally, there are **low-level** APIs for *library and framework developers* to:

- create and manage [event loops](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio-event-loop), which provide asynchronous APIs for [networking](https://docs.python.org/3/library/asyncio-eventloop.html#loop-create-server), running [subprocesses](https://docs.python.org/3/library/asyncio-eventloop.html#loop-subprocess-exec), handling [OS signals](https://docs.python.org/3/library/asyncio-eventloop.html#loop-add-signal-handler), etc;
- implement efficient protocols using [transports](https://docs.python.org/3/library/asyncio-protocol.html#asyncio-transports-protocols);
- [bridge](https://docs.python.org/3/library/asyncio-future.html#asyncio-futures) callback-based libraries and code with async/await syntax.

[Availability](https://docs.python.org/3/library/intro.html#availability): not WASI.

This module does not work or is not available on WebAssembly. See [WebAssembly platforms](https://docs.python.org/3/library/intro.html#wasm-availability) for more information.

asyncio REPL

You can experiment with an `asyncio` concurrent context in the [REPL](https://docs.python.org/3/glossary.html#term-REPL):

```
$ python -m asyncio
asyncio REPL ...
Use "await" directly instead of "asyncio.run()".
Type "help", "copyright", "credits" or "license" for more information.
>>> import asyncio
>>> await asyncio.sleep(10, result='hello')
'hello'
```

This REPL provides limited compatibility with [`PYTHON_BASIC_REPL`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHON_BASIC_REPL). It is recommended that the default REPL is used for full functionality and the latest features.

Raises an [auditing event](https://docs.python.org/3/library/sys.html#auditing) `cpython.run_stdin` with no arguments.

Changed in version 3.12.5: (also 3.11.10, 3.10.15, 3.9.20, and 3.8.20) Emits audit events.

Changed in version 3.13: Uses PyREPL if possible, in which case [`PYTHONSTARTUP`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONSTARTUP) is also executed. Emits audit events.

Reference

Guides and Tutorials

Note

The source code for asyncio can be found in [Lib/asyncio/](https://github.com/python/cpython/tree/3.14/Lib/asyncio/).