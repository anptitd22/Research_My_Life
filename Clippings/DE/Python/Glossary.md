---
title: "Glossary"
source: "https://docs.python.org/3/glossary.html#term-global-interpreter-lock"
author:
published:
created: 2026-04-11
description: ">>>, The default Python prompt of the interactive shell. Often seen for code examples which can be executed interactively in the interpreter.,,..., Can refer to:- The default Python prompt..."
tags:
  - "clippings"
---
## Glossary

`>>>` [¶](#term-0 "Link to this term")

The default Python prompt of the shell. Often seen for code examples which can be executed interactively in the interpreter.

`...`[¶](#term-... "Link to this term")

Can refer to:

- The default Python prompt of the shell when entering the code for an indented code block, when within a pair of matching left and right delimiters (parentheses, square brackets, curly braces or triple quotes), or after specifying a decorator.
- The three dots form of the [Ellipsis](https://docs.python.org/3/library/stdtypes.html#bltin-ellipsis-object) object.

abstract base class [¶](#term-abstract-base-class "Link to this term")

Abstract base classes complement by providing a way to define interfaces when other techniques like [`hasattr()`](https://docs.python.org/3/library/functions.html#hasattr "hasattr") would be clumsy or subtly wrong (for example with [magic methods](https://docs.python.org/3/reference/datamodel.html#special-lookup)). ABCs introduce virtual subclasses, which are classes that don’t inherit from a class but are still recognized by [`isinstance()`](https://docs.python.org/3/library/functions.html#isinstance "isinstance") and [`issubclass()`](https://docs.python.org/3/library/functions.html#issubclass "issubclass"); see the [`abc`](https://docs.python.org/3/library/abc.html#module-abc "abc: Abstract base classes according to :pep:`3119`.") module documentation. Python comes with many built-in ABCs for data structures (in the [`collections.abc`](https://docs.python.org/3/library/collections.abc.html#module-collections.abc "collections.abc: Abstract base classes for containers") module), numbers (in the [`numbers`](https://docs.python.org/3/library/numbers.html#module-numbers "numbers: Numeric abstract base classes (Complex, Real, Integral, etc.).") module), streams (in the [`io`](https://docs.python.org/3/library/io.html#module-io "io: Core tools for working with streams.") module), import finders and loaders (in the [`importlib.abc`](https://docs.python.org/3/library/importlib.html#module-importlib.abc "importlib.abc: Abstract base classes related to import") module). You can create your own ABCs with the `abc` module.

annotate function [¶](#term-annotate-function "Link to this term")

A function that can be called to retrieve the of an object. This function is accessible as the [`__annotate__`](https://docs.python.org/3/reference/datamodel.html#object.__annotate__ "object.__annotate__") attribute of functions, classes, and modules. Annotate functions are a subset of.

annotation [¶](#term-annotation "Link to this term")

A label associated with a variable, a class attribute or a function parameter or return value, used by convention as a.

Annotations of local variables cannot be accessed at runtime, but annotations of global variables, class attributes, and functions can be retrieved by calling [`annotationlib.get_annotations()`](https://docs.python.org/3/library/annotationlib.html#annotationlib.get_annotations "annotationlib.get_annotations") on modules, classes, and functions, respectively.

See,, [**PEP 484**](https://peps.python.org/pep-0484/), [**PEP 526**](https://peps.python.org/pep-0526/), and [**PEP 649**](https://peps.python.org/pep-0649/), which describe this functionality. Also see [Annotations Best Practices](https://docs.python.org/3/howto/annotations.html#annotations-howto) for best practices on working with annotations.

argument [¶](#term-argument "Link to this term")

A value passed to a (or ) when calling the function. There are two kinds of argument:

- *keyword argument*: an argument preceded by an identifier (e.g. `name=`) in a function call or passed as a value in a dictionary preceded by `**`. For example, `3` and `5` are both keyword arguments in the following calls to [`complex()`](https://docs.python.org/3/library/functions.html#complex "complex"):
	```
	complex(real=3, imag=5)
	complex(**{'real': 3, 'imag': 5})
	```
- *positional argument*: an argument that is not a keyword argument. Positional arguments can appear at the beginning of an argument list and/or be passed as elements of an preceded by `*`. For example, `3` and `5` are both positional arguments in the following calls:
	```
	complex(3, 5)
	complex(*(3, 5))
	```

Arguments are assigned to the named local variables in a function body. See the [Calls](https://docs.python.org/3/reference/expressions.html#calls) section for the rules governing this assignment. Syntactically, any expression can be used to represent an argument; the evaluated value is assigned to the local variable.

See also the glossary entry, the FAQ question on [the difference between arguments and parameters](https://docs.python.org/3/faq/programming.html#faq-argument-vs-parameter), and [**PEP 362**](https://peps.python.org/pep-0362/).

asynchronous context manager [¶](#term-asynchronous-context-manager "Link to this term")

An object which controls the environment seen in an [`async with`](https://docs.python.org/3/reference/compound_stmts.html#async-with) statement by defining [`__aenter__()`](https://docs.python.org/3/reference/datamodel.html#object.__aenter__ "object.__aenter__") and [`__aexit__()`](https://docs.python.org/3/reference/datamodel.html#object.__aexit__ "object.__aexit__") methods. Introduced by [**PEP 492**](https://peps.python.org/pep-0492/).

asynchronous generator [¶](#term-asynchronous-generator "Link to this term")

A function which returns an. It looks like a coroutine function defined with [`async def`](https://docs.python.org/3/reference/compound_stmts.html#async-def) except that it contains [`yield`](https://docs.python.org/3/reference/simple_stmts.html#yield) expressions for producing a series of values usable in an [`async for`](https://docs.python.org/3/reference/compound_stmts.html#async-for) loop.

Usually refers to an asynchronous generator function, but may refer to an *asynchronous generator iterator* in some contexts. In cases where the intended meaning isn’t clear, using the full terms avoids ambiguity.

An asynchronous generator function may contain [`await`](https://docs.python.org/3/reference/expressions.html#await) expressions as well as [`async for`](https://docs.python.org/3/reference/compound_stmts.html#async-for), and [`async with`](https://docs.python.org/3/reference/compound_stmts.html#async-with) statements.

asynchronous generator iterator [¶](#term-asynchronous-generator-iterator "Link to this term")

An object created by an function.

This is an which when called using the [`__anext__()`](https://docs.python.org/3/reference/datamodel.html#object.__anext__ "object.__anext__") method returns an awaitable object which will execute the body of the asynchronous generator function until the next [`yield`](https://docs.python.org/3/reference/simple_stmts.html#yield) expression.

Each [`yield`](https://docs.python.org/3/reference/simple_stmts.html#yield) temporarily suspends processing, remembering the execution state (including local variables and pending try-statements). When the *asynchronous generator iterator* effectively resumes with another awaitable returned by [`__anext__()`](https://docs.python.org/3/reference/datamodel.html#object.__anext__ "object.__anext__"), it picks up where it left off. See [**PEP 492**](https://peps.python.org/pep-0492/) and [**PEP 525**](https://peps.python.org/pep-0525/).

asynchronous iterable [¶](#term-asynchronous-iterable "Link to this term")

An object, that can be used in an [`async for`](https://docs.python.org/3/reference/compound_stmts.html#async-for) statement. Must return an from its [`__aiter__()`](https://docs.python.org/3/reference/datamodel.html#object.__aiter__ "object.__aiter__") method. Introduced by [**PEP 492**](https://peps.python.org/pep-0492/).

asynchronous iterator [¶](#term-asynchronous-iterator "Link to this term")

An object that implements the [`__aiter__()`](https://docs.python.org/3/reference/datamodel.html#object.__aiter__ "object.__aiter__") and [`__anext__()`](https://docs.python.org/3/reference/datamodel.html#object.__anext__ "object.__anext__") methods. `__anext__()` must return an object. [`async for`](https://docs.python.org/3/reference/compound_stmts.html#async-for) resolves the awaitables returned by an asynchronous iterator’s `__anext__()` method until it raises a [`StopAsyncIteration`](https://docs.python.org/3/library/exceptions.html#StopAsyncIteration "StopAsyncIteration") exception. Introduced by [**PEP 492**](https://peps.python.org/pep-0492/).

atomic operation [¶](#term-atomic-operation "Link to this term")

An operation that appears to execute as a single, indivisible step: no other thread can observe it half-done, and its effects become visible all at once. Python does not guarantee that high-level statements are atomic (for example, `x += 1` performs multiple bytecode operations and is not atomic). Atomicity is only guaranteed where explicitly documented. See also and.

attached thread state [¶](#term-attached-thread-state "Link to this term")

A that is active for the current OS thread.

When a is attached, the OS thread has access to the full Python C API and can safely invoke the bytecode interpreter.

Unless a function explicitly notes otherwise, attempting to call the C API without an attached thread state will result in a fatal error or undefined behavior. A thread state can be attached and detached explicitly by the user through the C API, or implicitly by the runtime, including during blocking C calls and by the bytecode interpreter in between calls.

On most builds of Python, having an attached thread state implies that the caller holds the for the current interpreter, so only one OS thread can have an attached thread state at a given moment. In of Python, threads can concurrently hold an attached thread state, allowing for true parallelism of the bytecode interpreter.

attribute [¶](#term-attribute "Link to this term")

A value associated with an object which is usually referenced by name using dotted expressions. For example, if an object *o* has an attribute *a* it would be referenced as *o.a*.

It is possible to give an object an attribute whose name is not an identifier as defined by [Names (identifiers and keywords)](https://docs.python.org/3/reference/lexical_analysis.html#identifiers), for example using [`setattr()`](https://docs.python.org/3/library/functions.html#setattr "setattr"), if the object allows it. Such an attribute will not be accessible using a dotted expression, and would instead need to be retrieved with [`getattr()`](https://docs.python.org/3/library/functions.html#getattr "getattr").

awaitable [¶](#term-awaitable "Link to this term")

An object that can be used in an [`await`](https://docs.python.org/3/reference/expressions.html#await) expression. Can be a or an object with an [`__await__()`](https://docs.python.org/3/reference/datamodel.html#object.__await__ "object.__await__") method. See also [**PEP 492**](https://peps.python.org/pep-0492/).

BDFL [¶](#term-BDFL "Link to this term")

Benevolent Dictator For Life, a.k.a. [Guido van Rossum](https://gvanrossum.github.io/), Python’s creator.

binary file [¶](#term-binary-file "Link to this term")

A able to read and write. Examples of binary files are files opened in binary mode (`'rb'`, `'wb'` or `'rb+'`), [`sys.stdin.buffer`](https://docs.python.org/3/library/sys.html#sys.stdin "sys.stdin"), [`sys.stdout.buffer`](https://docs.python.org/3/library/sys.html#sys.stdout "sys.stdout"), and instances of [`io.BytesIO`](https://docs.python.org/3/library/io.html#io.BytesIO "io.BytesIO") and [`gzip.GzipFile`](https://docs.python.org/3/library/gzip.html#gzip.GzipFile "gzip.GzipFile").

See also for a file object able to read and write [`str`](https://docs.python.org/3/library/stdtypes.html#str "str") objects.

borrowed reference [¶](#term-borrowed-reference "Link to this term")

In Python’s C API, a borrowed reference is a reference to an object, where the code using the object does not own the reference. It becomes a dangling pointer if the object is destroyed. For example, a garbage collection can remove the last to the object and so destroy it.

Calling [`Py_INCREF()`](https://docs.python.org/3/c-api/refcounting.html#c.Py_INCREF "Py_INCREF") on the is recommended to convert it to a in-place, except when the object cannot be destroyed before the last usage of the borrowed reference. The [`Py_NewRef()`](https://docs.python.org/3/c-api/refcounting.html#c.Py_NewRef "Py_NewRef") function can be used to create a new strong reference.

bytes-like object [¶](#term-bytes-like-object "Link to this term")

An object that supports the [Buffer Protocol](https://docs.python.org/3/c-api/buffer.html#bufferobjects) and can export a C- buffer. This includes all [`bytes`](https://docs.python.org/3/library/stdtypes.html#bytes "bytes"), [`bytearray`](https://docs.python.org/3/library/stdtypes.html#bytearray "bytearray"), and [`array.array`](https://docs.python.org/3/library/array.html#array.array "array.array") objects, as well as many common [`memoryview`](https://docs.python.org/3/library/stdtypes.html#memoryview "memoryview") objects. Bytes-like objects can be used for various operations that work with binary data; these include compression, saving to a binary file, and sending over a socket.

Some operations need the binary data to be mutable. The documentation often refers to these as “read-write bytes-like objects”. Example mutable buffer objects include [`bytearray`](https://docs.python.org/3/library/stdtypes.html#bytearray "bytearray") and a [`memoryview`](https://docs.python.org/3/library/stdtypes.html#memoryview "memoryview") of a `bytearray`. Other operations require the binary data to be stored in immutable objects (“read-only bytes-like objects”); examples of these include [`bytes`](https://docs.python.org/3/library/stdtypes.html#bytes "bytes") and a `memoryview` of a `bytes` object.

bytecode [¶](#term-bytecode "Link to this term")

Python source code is compiled into bytecode, the internal representation of a Python program in the CPython interpreter. The bytecode is also cached in `.pyc` files so that executing the same file is faster the second time (recompilation from source to bytecode can be avoided). This “intermediate language” is said to run on a that executes the machine code corresponding to each bytecode. Do note that bytecodes are not expected to work between different Python virtual machines, nor to be stable between Python releases.

A list of bytecode instructions can be found in the documentation for [the dis module](https://docs.python.org/3/library/dis.html#bytecodes).

callable [¶](#term-callable "Link to this term")

A callable is an object that can be called, possibly with a set of arguments (see ), with the following syntax:

```
callable(argument1, argument2, argumentN)
```

A, and by extension a, is a callable. An instance of a class that implements the [`__call__()`](https://docs.python.org/3/reference/datamodel.html#object.__call__ "object.__call__") method is also a callable.

callback [¶](#term-callback "Link to this term")

A subroutine function which is passed as an argument to be executed at some point in the future.

class [¶](#term-class "Link to this term")

A template for creating user-defined objects. Class definitions normally contain method definitions which operate on instances of the class.

class variable [¶](#term-class-variable "Link to this term")

A variable defined in a class and intended to be modified only at class level (i.e., not in an instance of the class).

closure variable [¶](#term-closure-variable "Link to this term")

A referenced from a that is defined in an outer scope rather than being resolved at runtime from the globals or builtin namespaces. May be explicitly defined with the [`nonlocal`](https://docs.python.org/3/reference/simple_stmts.html#nonlocal) keyword to allow write access, or implicitly defined if the variable is only being read.

For example, in the `inner` function in the following code, both `x` and `print` are, but only `x` is a *closure variable*:

```
def outer():
    x = 0
    def inner():
        nonlocal x
        x += 1
        print(x)
    return inner
```

Due to the [`codeobject.co_freevars`](https://docs.python.org/3/reference/datamodel.html#codeobject.co_freevars "codeobject.co_freevars") attribute (which, despite its name, only includes the names of closure variables rather than listing all referenced free variables), the more general term is sometimes used even when the intended meaning is to refer specifically to closure variables.

complex number [¶](#term-complex-number "Link to this term")

An extension of the familiar real number system in which all numbers are expressed as a sum of a real part and an imaginary part. Imaginary numbers are real multiples of the imaginary unit (the square root of `-1`), often written `i` in mathematics or `j` in engineering. Python has built-in support for complex numbers, which are written with this latter notation; the imaginary part is written with a `j` suffix, e.g., `3+1j`. To get access to complex equivalents of the [`math`](https://docs.python.org/3/library/math.html#module-math "math: Mathematical functions (sin() etc.).") module, use [`cmath`](https://docs.python.org/3/library/cmath.html#module-cmath "cmath: Mathematical functions for complex numbers."). Use of complex numbers is a fairly advanced mathematical feature. If you’re not aware of a need for them, it’s almost certain you can safely ignore them.

concurrency [¶](#term-concurrency "Link to this term")

The ability of a computer program to perform multiple tasks at the same time. Python provides libraries for writing programs that make use of different forms of concurrency. [`asyncio`](https://docs.python.org/3/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O.") is a library for dealing with asynchronous tasks and coroutines. [`threading`](https://docs.python.org/3/library/threading.html#module-threading "threading: Thread-based parallelism.") provides access to operating system threads and [`multiprocessing`](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing "multiprocessing: Process-based parallelism.") to operating system processes. Multi-core processors can execute threads and processes on different CPU cores at the same time (see ).

concurrent modification [¶](#term-concurrent-modification "Link to this term")

When multiple threads modify shared data at the same time. Concurrent modification without proper synchronization can cause, and might also trigger a, data corruption, or both.

context [¶](#term-context "Link to this term")

This term has different meanings depending on where and how it is used. Some common meanings:

- The temporary state or environment established by a via a [`with`](https://docs.python.org/3/reference/compound_stmts.html#with) statement.
- The collection of key­value bindings associated with a particular [`contextvars.Context`](https://docs.python.org/3/library/contextvars.html#contextvars.Context "contextvars.Context") object and accessed via [`ContextVar`](https://docs.python.org/3/library/contextvars.html#contextvars.ContextVar "contextvars.ContextVar") objects. Also see.
- A [`contextvars.Context`](https://docs.python.org/3/library/contextvars.html#contextvars.Context "contextvars.Context") object. Also see.

context management protocol [¶](#term-context-management-protocol "Link to this term")

The [`__enter__()`](https://docs.python.org/3/reference/datamodel.html#object.__enter__ "object.__enter__") and [`__exit__()`](https://docs.python.org/3/reference/datamodel.html#object.__exit__ "object.__exit__") methods called by the [`with`](https://docs.python.org/3/reference/compound_stmts.html#with) statement. See [**PEP 343**](https://peps.python.org/pep-0343/).

context manager [¶](#term-context-manager "Link to this term")

An object which implements the and controls the environment seen in a [`with`](https://docs.python.org/3/reference/compound_stmts.html#with) statement. See [**PEP 343**](https://peps.python.org/pep-0343/).

context variable [¶](#term-context-variable "Link to this term")

A variable whose value depends on which context is the. Values are accessed via [`contextvars.ContextVar`](https://docs.python.org/3/library/contextvars.html#contextvars.ContextVar "contextvars.ContextVar") objects. Context variables are primarily used to isolate state between concurrent asynchronous tasks.

contiguous [¶](#term-contiguous "Link to this term")

A buffer is considered contiguous exactly if it is either *C-contiguous* or *Fortran contiguous*. Zero-dimensional buffers are C and Fortran contiguous. In one-dimensional arrays, the items must be laid out in memory next to each other, in order of increasing indexes starting from zero. In multidimensional C-contiguous arrays, the last index varies the fastest when visiting items in order of memory address. However, in Fortran contiguous arrays, the first index varies the fastest.

coroutine [¶](#term-coroutine "Link to this term")

Coroutines are a more generalized form of subroutines. Subroutines are entered at one point and exited at another point. Coroutines can be entered, exited, and resumed at many different points. They can be implemented with the [`async def`](https://docs.python.org/3/reference/compound_stmts.html#async-def) statement. See also [**PEP 492**](https://peps.python.org/pep-0492/).

coroutine function [¶](#term-coroutine-function "Link to this term")

A function which returns a object. A coroutine function may be defined with the [`async def`](https://docs.python.org/3/reference/compound_stmts.html#async-def) statement, and may contain [`await`](https://docs.python.org/3/reference/expressions.html#await), [`async for`](https://docs.python.org/3/reference/compound_stmts.html#async-for), and [`async with`](https://docs.python.org/3/reference/compound_stmts.html#async-with) keywords. These were introduced by [**PEP 492**](https://peps.python.org/pep-0492/).

CPython [¶](#term-CPython "Link to this term")

The canonical implementation of the Python programming language, as distributed on [python.org](https://www.python.org/). The term “CPython” is used when necessary to distinguish this implementation from others such as Jython or IronPython.

current context [¶](#term-current-context "Link to this term")

The ([`contextvars.Context`](https://docs.python.org/3/library/contextvars.html#contextvars.Context "contextvars.Context") object) that is currently used by [`ContextVar`](https://docs.python.org/3/library/contextvars.html#contextvars.ContextVar "contextvars.ContextVar") objects to access (get or set) the values of. Each thread has its own current context. Frameworks for executing asynchronous tasks (see [`asyncio`](https://docs.python.org/3/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O.")) associate each task with a context which becomes the current context whenever the task starts or resumes execution.

cyclic isolate [¶](#term-cyclic-isolate "Link to this term")

A subgroup of one or more objects that reference each other in a reference cycle, but are not referenced by objects outside the group. The goal of the is to identify these groups and break the reference cycles so that the memory can be reclaimed.

data race [¶](#term-data-race "Link to this term")

A situation where multiple threads access the same memory location concurrently, at least one of the accesses is a write, and the threads do not use any synchronization to control their access. Data races lead to behavior and can cause data corruption. Proper use of and other prevents data races. Note that data races can only happen in native code, but that might be exposed in a Python API. See also and.

deadlock [¶](#term-deadlock "Link to this term")

A situation in which two or more tasks (threads, processes, or coroutines) wait indefinitely for each other to release resources or complete actions, preventing any from making progress. For example, if thread A holds lock 1 and waits for lock 2, while thread B holds lock 2 and waits for lock 1, both threads will wait indefinitely. In Python this often arises from acquiring multiple locks in conflicting orders or from circular join/await dependencies. Deadlocks can be avoided by always acquiring multiple in a consistent order. See also lock and.

decorator [¶](#term-decorator "Link to this term")

A function returning another function, usually applied as a function transformation using the `@wrapper` syntax. Common examples for decorators are [`classmethod()`](https://docs.python.org/3/library/functions.html#classmethod "classmethod") and [`staticmethod()`](https://docs.python.org/3/library/functions.html#staticmethod "staticmethod").

The decorator syntax is merely syntactic sugar, the following two function definitions are semantically equivalent:

```
def f(arg):
    ...
f = staticmethod(f)

@staticmethod
def f(arg):
    ...
```

The same concept exists for classes, but is less commonly used there. See the documentation for [function definitions](https://docs.python.org/3/reference/compound_stmts.html#function) and [class definitions](https://docs.python.org/3/reference/compound_stmts.html#class) for more about decorators.

descriptor [¶](#term-descriptor "Link to this term")

Any object which defines the methods [`__get__()`](https://docs.python.org/3/reference/datamodel.html#object.__get__ "object.__get__"), [`__set__()`](https://docs.python.org/3/reference/datamodel.html#object.__set__ "object.__set__"), or [`__delete__()`](https://docs.python.org/3/reference/datamodel.html#object.__delete__ "object.__delete__"). When a class attribute is a descriptor, its special binding behavior is triggered upon attribute lookup. Normally, using *a.b* to get, set or delete an attribute looks up the object named *b* in the class dictionary for *a*, but if *b* is a descriptor, the respective descriptor method gets called. Understanding descriptors is a key to a deep understanding of Python because they are the basis for many features including functions, methods, properties, class methods, static methods, and reference to super classes.

For more information about descriptors’ methods, see [Implementing Descriptors](https://docs.python.org/3/reference/datamodel.html#descriptors) or the [Descriptor How To Guide](https://docs.python.org/3/howto/descriptor.html#descriptorhowto).

dictionary [¶](#term-dictionary "Link to this term")

An associative array, where arbitrary keys are mapped to values. The keys can be any object with [`__hash__()`](https://docs.python.org/3/reference/datamodel.html#object.__hash__ "object.__hash__") and [`__eq__()`](https://docs.python.org/3/reference/datamodel.html#object.__eq__ "object.__eq__") methods. Called a hash in Perl.

dictionary comprehension [¶](#term-dictionary-comprehension "Link to this term")

A compact way to process all or part of the elements in an iterable and return a dictionary with the results. `results = {n: n ** 2 for n in range(10)}` generates a dictionary containing key `n` mapped to value `n ** 2`. See [Displays for lists, sets and dictionaries](https://docs.python.org/3/reference/expressions.html#comprehensions).

dictionary view [¶](#term-dictionary-view "Link to this term")

The objects returned from [`dict.keys()`](https://docs.python.org/3/library/stdtypes.html#dict.keys "dict.keys"), [`dict.values()`](https://docs.python.org/3/library/stdtypes.html#dict.values "dict.values"), and [`dict.items()`](https://docs.python.org/3/library/stdtypes.html#dict.items "dict.items") are called dictionary views. They provide a dynamic view on the dictionary’s entries, which means that when the dictionary changes, the view reflects these changes. To force the dictionary view to become a full list use `list(dictview)`. See [Dictionary view objects](https://docs.python.org/3/library/stdtypes.html#dict-views).

docstring [¶](#term-docstring "Link to this term")

A string literal which appears as the first expression in a class, function or module. While ignored when the suite is executed, it is recognized by the compiler and put into the [`__doc__`](https://docs.python.org/3/library/stdtypes.html#definition.__doc__ "definition.__doc__") attribute of the enclosing class, function or module. Since it is available via introspection, it is the canonical place for documentation of the object.

duck-typing [¶](#term-duck-typing "Link to this term")

A programming style which does not look at an object’s type to determine if it has the right interface; instead, the method or attribute is simply called or used (“If it looks like a duck and quacks like a duck, it must be a duck.”) By emphasizing interfaces rather than specific types, well-designed code improves its flexibility by allowing polymorphic substitution. Duck-typing avoids tests using [`type()`](https://docs.python.org/3/library/functions.html#type "type") or [`isinstance()`](https://docs.python.org/3/library/functions.html#isinstance "isinstance"). (Note, however, that duck-typing can be complemented with.) Instead, it typically employs [`hasattr()`](https://docs.python.org/3/library/functions.html#hasattr "hasattr") tests or programming.

dunder [¶](#term-dunder "Link to this term")

An informal short-hand for “double underscore”, used when talking about a. For example, `__init__` is often pronounced “dunder init”.

EAFP [¶](#term-EAFP "Link to this term")

Easier to ask for forgiveness than permission. This common Python coding style assumes the existence of valid keys or attributes and catches exceptions if the assumption proves false. This clean and fast style is characterized by the presence of many [`try`](https://docs.python.org/3/reference/compound_stmts.html#try) and [`except`](https://docs.python.org/3/reference/compound_stmts.html#except) statements. The technique contrasts with the style common to many other languages such as C.

evaluate function [¶](#term-evaluate-function "Link to this term")

A function that can be called to evaluate a lazily evaluated attribute of an object, such as the value of type aliases created with the [`type`](https://docs.python.org/3/reference/simple_stmts.html#type) statement.

expression [¶](#term-expression "Link to this term")

A piece of syntax which can be evaluated to some value. In other words, an expression is an accumulation of expression elements like literals, names, attribute access, operators or function calls which all return a value. In contrast to many other languages, not all language constructs are expressions. There are also s which cannot be used as expressions, such as [`while`](https://docs.python.org/3/reference/compound_stmts.html#while). Assignments are also statements, not expressions.

extension module [¶](#term-extension-module "Link to this term")

A module written in C or C++, using Python’s C API to interact with the core and with user code.

f-string [¶](#term-f-string "Link to this term")

f-strings [¶](#term-f-strings "Link to this term")

String literals prefixed with `f` or `F` are commonly called “f-strings” which is short for [formatted string literals](https://docs.python.org/3/reference/lexical_analysis.html#f-strings). See also [**PEP 498**](https://peps.python.org/pep-0498/).

file object [¶](#term-file-object "Link to this term")

An object exposing a file-oriented API (with methods such as `read()` or `write()`) to an underlying resource. Depending on the way it was created, a file object can mediate access to a real on-disk file or to another type of storage or communication device (for example standard input/output, in-memory buffers, sockets, pipes, etc.). File objects are also called *file-like objects* or *streams*.

There are actually three categories of file objects: raw, buffered binary files and. Their interfaces are defined in the [`io`](https://docs.python.org/3/library/io.html#module-io "io: Core tools for working with streams.") module. The canonical way to create a file object is by using the [`open()`](https://docs.python.org/3/library/functions.html#open "open") function.

file-like object [¶](#term-file-like-object "Link to this term")

A synonym for.

Encoding and error handler used by Python to decode bytes from the operating system and encode Unicode to the operating system.

The filesystem encoding must guarantee to successfully decode all bytes below 128. If the file system encoding fails to provide this guarantee, API functions can raise [`UnicodeError`](https://docs.python.org/3/library/exceptions.html#UnicodeError "UnicodeError").

The [`sys.getfilesystemencoding()`](https://docs.python.org/3/library/sys.html#sys.getfilesystemencoding "sys.getfilesystemencoding") and [`sys.getfilesystemencodeerrors()`](https://docs.python.org/3/library/sys.html#sys.getfilesystemencodeerrors "sys.getfilesystemencodeerrors") functions can be used to get the filesystem encoding and error handler.

The are configured at Python startup by the [`PyConfig_Read()`](https://docs.python.org/3/c-api/init_config.html#c.PyConfig_Read "PyConfig_Read") function: see [`filesystem_encoding`](https://docs.python.org/3/c-api/init_config.html#c.PyConfig.filesystem_encoding "PyConfig.filesystem_encoding") and [`filesystem_errors`](https://docs.python.org/3/c-api/init_config.html#c.PyConfig.filesystem_errors "PyConfig.filesystem_errors") members of [`PyConfig`](https://docs.python.org/3/c-api/init_config.html#c.PyConfig "PyConfig").

See also the.

finder [¶](#term-finder "Link to this term")

An object that tries to find the for a module that is being imported.

There are two types of finder: for use with [`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path"), and for use with [`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks").

See [Finders and loaders](https://docs.python.org/3/reference/import.html#finders-and-loaders) and [`importlib`](https://docs.python.org/3/library/importlib.html#module-importlib "importlib: The implementation of the import machinery.") for much more detail.

floor division [¶](#term-floor-division "Link to this term")

Mathematical division that rounds down to nearest integer. The floor division operator is `//`. For example, the expression `11 // 4` evaluates to `2` in contrast to the `2.75` returned by float true division. Note that `(-11) // 4` is `-3` because that is `-2.75` rounded *downward*. See [**PEP 238**](https://peps.python.org/pep-0238/).

free threading [¶](#term-free-threading "Link to this term")

A threading model where multiple threads can run Python bytecode simultaneously within the same interpreter. This is in contrast to the which allows only one thread to execute Python bytecode at a time. See [**PEP 703**](https://peps.python.org/pep-0703/).

free-threaded build [¶](#term-free-threaded-build "Link to this term")

A build of that supports, configured using the [`--disable-gil`](https://docs.python.org/3/using/configure.html#cmdoption-disable-gil) option before compilation.

See [Python support for free threading](https://docs.python.org/3/howto/free-threading-python.html#freethreading-python-howto).

free variable [¶](#term-free-variable "Link to this term")

Formally, as defined in the [language execution model](https://docs.python.org/3/reference/executionmodel.html#bind-names), a free variable is any variable used in a namespace which is not a local variable in that namespace. See for an example. Pragmatically, due to the name of the [`codeobject.co_freevars`](https://docs.python.org/3/reference/datamodel.html#codeobject.co_freevars "codeobject.co_freevars") attribute, the term is also sometimes used as a synonym for closure variable.

function [¶](#term-function "Link to this term")

A series of statements which returns some value to a caller. It can also be passed zero or more which may be used in the execution of the body. See also,, and the [Function definitions](https://docs.python.org/3/reference/compound_stmts.html#function) section.

function annotation [¶](#term-function-annotation "Link to this term")

An of a function parameter or return value.

Function annotations are usually used for: for example, this function is expected to take two [`int`](https://docs.python.org/3/library/functions.html#int "int") arguments and is also expected to have an `int` return value:

```
def sum_two_numbers(a: int, b: int) -> int:
   return a + b
```

Function annotation syntax is explained in section [Function definitions](https://docs.python.org/3/reference/compound_stmts.html#function).

See and [**PEP 484**](https://peps.python.org/pep-0484/), which describe this functionality. Also see [Annotations Best Practices](https://docs.python.org/3/howto/annotations.html#annotations-howto) for best practices on working with annotations.

\_\_future\_\_ [¶](#term-__future__ "Link to this term")

A [future statement](https://docs.python.org/3/reference/simple_stmts.html#future), `from __future__ import <feature>`, directs the compiler to compile the current module using syntax or semantics that will become standard in a future release of Python. The [`__future__`](https://docs.python.org/3/library/__future__.html#module-__future__ "__future__: Future statement definitions") module documents the possible values of *feature*. By importing this module and evaluating its variables, you can see when a new feature was first added to the language and when it will (or did) become the default:

```
>>> import __future__
>>> __future__.division
_Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 8192)
```

garbage collection [¶](#term-garbage-collection "Link to this term")

The process of freeing memory when it is not used anymore. Python performs garbage collection via reference counting and a cyclic garbage collector that is able to detect and break reference cycles. The garbage collector can be controlled using the [`gc`](https://docs.python.org/3/library/gc.html#module-gc "gc: Interface to the cycle-detecting garbage collector.") module.

generator [¶](#term-generator "Link to this term")

A function which returns a. It looks like a normal function except that it contains [`yield`](https://docs.python.org/3/reference/simple_stmts.html#yield) expressions for producing a series of values usable in a for-loop or that can be retrieved one at a time with the [`next()`](https://docs.python.org/3/library/functions.html#next "next") function.

Usually refers to a generator function, but may refer to a *generator iterator* in some contexts. In cases where the intended meaning isn’t clear, using the full terms avoids ambiguity.

generator iterator [¶](#term-generator-iterator "Link to this term")

An object created by a function.

Each [`yield`](https://docs.python.org/3/reference/simple_stmts.html#yield) temporarily suspends processing, remembering the execution state (including local variables and pending try-statements). When the *generator iterator* resumes, it picks up where it left off (in contrast to functions which start fresh on every invocation).

generator expression [¶](#term-generator-expression "Link to this term")

An that returns an. It looks like a normal expression followed by a `for` clause defining a loop variable, range, and an optional `if` clause. The combined expression generates values for an enclosing function:

```
>>> sum(i*i for i in range(10))         # sum of squares 0, 1, 4, ... 81
285
```

generic function [¶](#term-generic-function "Link to this term")

A function composed of multiple functions implementing the same operation for different types. Which implementation should be used during a call is determined by the dispatch algorithm.

See also the glossary entry, the [`functools.singledispatch()`](https://docs.python.org/3/library/functools.html#functools.singledispatch "functools.singledispatch") decorator, and [**PEP 443**](https://peps.python.org/pep-0443/).

generic type [¶](#term-generic-type "Link to this term")

A that can be parameterized; typically a [container class](https://docs.python.org/3/reference/datamodel.html#sequence-types) such as [`list`](https://docs.python.org/3/library/stdtypes.html#list "list") or [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict"). Used for and.

For more details, see [generic alias types](https://docs.python.org/3/library/stdtypes.html#types-genericalias), [**PEP 483**](https://peps.python.org/pep-0483/), [**PEP 484**](https://peps.python.org/pep-0484/), [**PEP 585**](https://peps.python.org/pep-0585/), and the [`typing`](https://docs.python.org/3/library/typing.html#module-typing "typing: Support for type hints (see :pep:`484`).") module.

GIL [¶](#term-GIL "Link to this term")

See.

global interpreter lock [¶](#term-global-interpreter-lock "Link to this term")

The mechanism used by the interpreter to assure that only one thread executes Python at a time. This simplifies the CPython implementation by making the object model (including critical built-in types such as [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict")) implicitly safe against concurrent access. Locking the entire interpreter makes it easier for the interpreter to be multi-threaded, at the expense of much of the parallelism afforded by multi-processor machines.

However, some extension modules, either standard or third-party, are designed so as to release the GIL when doing computationally intensive tasks such as compression or hashing. Also, the GIL is always released when doing I/O.

As of Python 3.13, the GIL can be disabled using the [`--disable-gil`](https://docs.python.org/3/using/configure.html#cmdoption-disable-gil) build configuration. After building Python with this option, code must be run with [`-X gil=0`](https://docs.python.org/3/using/cmdline.html#cmdoption-X) or after setting the [`PYTHON_GIL=0`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHON_GIL) environment variable. This feature enables improved performance for multi-threaded applications and makes it easier to use multi-core CPUs efficiently. For more details, see [**PEP 703**](https://peps.python.org/pep-0703/).

In prior versions of Python’s C API, a function might declare that it requires the GIL to be held in order to use it. This refers to having an.

global state [¶](#term-global-state "Link to this term")

Data that is accessible throughout a program, such as module-level variables, class variables, or C static variables in. In multi-threaded programs, global state shared between threads typically requires synchronization to avoid and.

hash-based pyc [¶](#term-hash-based-pyc "Link to this term")

A bytecode cache file that uses the hash rather than the last-modified time of the corresponding source file to determine its validity. See [Cached bytecode invalidation](https://docs.python.org/3/reference/import.html#pyc-invalidation).

hashable [¶](#term-hashable "Link to this term")

An object is *hashable* if it has a hash value which never changes during its lifetime (it needs a [`__hash__()`](https://docs.python.org/3/reference/datamodel.html#object.__hash__ "object.__hash__") method), and can be compared to other objects (it needs an [`__eq__()`](https://docs.python.org/3/reference/datamodel.html#object.__eq__ "object.__eq__") method). Hashable objects which compare equal must have the same hash value.

Hashability makes an object usable as a dictionary key and a set member, because these data structures use the hash value internally.

Most of Python’s immutable built-in objects are hashable; mutable containers (such as lists or dictionaries) are not; immutable containers (such as tuples and frozensets) are only hashable if their elements are hashable. Objects which are instances of user-defined classes are hashable by default. They all compare unequal (except with themselves), and their hash value is derived from their [`id()`](https://docs.python.org/3/library/functions.html#id "id").

IDLE [¶](#term-IDLE "Link to this term")

An Integrated Development and Learning Environment for Python. [IDLE — Python editor and shell](https://docs.python.org/3/library/idle.html#idle) is a basic editor and interpreter environment which ships with the standard distribution of Python.

immortal [¶](#term-immortal "Link to this term")

*Immortal objects* are a CPython implementation detail introduced in [**PEP 683**](https://peps.python.org/pep-0683/).

If an object is immortal, its is never modified, and therefore it is never deallocated while the interpreter is running. For example, [`True`](https://docs.python.org/3/library/constants.html#True "True") and [`None`](https://docs.python.org/3/library/constants.html#None "None") are immortal in CPython.

Immortal objects can be identified via [`sys._is_immortal()`](https://docs.python.org/3/library/sys.html#sys._is_immortal "sys._is_immortal"), or via [`PyUnstable_IsImmortal()`](https://docs.python.org/3/c-api/object.html#c.PyUnstable_IsImmortal "PyUnstable_IsImmortal") in the C API.

immutable [¶](#term-immutable "Link to this term")

An object with a fixed value. Immutable objects include numbers, strings and tuples. Such an object cannot be altered. A new object has to be created if a different value has to be stored. They play an important role in places where a constant hash value is needed, for example as a key in a dictionary. Immutable objects are inherently because their state cannot be modified after creation, eliminating concerns about improperly synchronized.

import path [¶](#term-import-path "Link to this term")

A list of locations (or ) that are searched by the for modules to import. During import, this list of locations usually comes from [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path"), but for subpackages it may also come from the parent package’s `__path__` attribute.

importing [¶](#term-importing "Link to this term")

The process by which Python code in one module is made available to Python code in another module.

importer [¶](#term-importer "Link to this term")

An object that both finds and loads a module; both a and object.

index [¶](#term-index "Link to this term")

A numeric value that represents the position of an element in a.

In Python, indexing starts at zero. For example, `things[0]` names the *first* element of `things`; `things[1]` names the second one.

In some contexts, Python allows negative indexes for counting from the end of a sequence, and indexing using.

See also.

interactive [¶](#term-interactive "Link to this term")

Python has an interactive interpreter which means you can enter statements and expressions at the interpreter prompt, immediately execute them and see their results. Just launch `python` with no arguments (possibly by selecting it from your computer’s main menu). It is a very powerful way to test out new ideas or inspect modules and packages (remember `help(x)`). For more on interactive mode, see [Interactive Mode](https://docs.python.org/3/tutorial/appendix.html#tut-interac).

interpreted [¶](#term-interpreted "Link to this term")

Python is an interpreted language, as opposed to a compiled one, though the distinction can be blurry because of the presence of the bytecode compiler. This means that source files can be run directly without explicitly creating an executable which is then run. Interpreted languages typically have a shorter development/debug cycle than compiled ones, though their programs generally also run more slowly. See also.

interpreter shutdown [¶](#term-interpreter-shutdown "Link to this term")

When asked to shut down, the Python interpreter enters a special phase where it gradually releases all allocated resources, such as modules and various critical internal structures. It also makes several calls to the. This can trigger the execution of code in user-defined destructors or weakref callbacks. Code executed during the shutdown phase can encounter various exceptions as the resources it relies on may not function anymore (common examples are library modules or the warnings machinery).

The main reason for interpreter shutdown is that the `__main__` module or the script being run has finished executing.

iterable [¶](#term-iterable "Link to this term")

An object capable of returning its members one at a time. Examples of iterables include all sequence types (such as [`list`](https://docs.python.org/3/library/stdtypes.html#list "list"), [`str`](https://docs.python.org/3/library/stdtypes.html#str "str"), and [`tuple`](https://docs.python.org/3/library/stdtypes.html#tuple "tuple")) and some non-sequence types like [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict"),, and objects of any classes you define with an [`__iter__()`](https://docs.python.org/3/reference/datamodel.html#object.__iter__ "object.__iter__") method or with a [`__getitem__()`](https://docs.python.org/3/reference/datamodel.html#object.__getitem__ "object.__getitem__") method that implements semantics.

Iterables can be used in a [`for`](https://docs.python.org/3/reference/compound_stmts.html#for) loop and in many other places where a sequence is needed ([`zip()`](https://docs.python.org/3/library/functions.html#zip "zip"), [`map()`](https://docs.python.org/3/library/functions.html#map "map"), …). When an iterable object is passed as an argument to the built-in function [`iter()`](https://docs.python.org/3/library/functions.html#iter "iter"), it returns an iterator for the object. This iterator is good for one pass over the set of values. When using iterables, it is usually not necessary to call `iter()` or deal with iterator objects yourself. The `for` statement does that automatically for you, creating a temporary unnamed variable to hold the iterator for the duration of the loop. See also,, and.

iterator [¶](#term-iterator "Link to this term")

An object representing a stream of data. Repeated calls to the iterator’s [`__next__()`](https://docs.python.org/3/library/stdtypes.html#iterator.__next__ "iterator.__next__") method (or passing it to the built-in function [`next()`](https://docs.python.org/3/library/functions.html#next "next")) return successive items in the stream. When no more data are available a [`StopIteration`](https://docs.python.org/3/library/exceptions.html#StopIteration "StopIteration") exception is raised instead. At this point, the iterator object is exhausted and any further calls to its `__next__()` method just raise `StopIteration` again. Iterators are required to have an [`__iter__()`](https://docs.python.org/3/library/stdtypes.html#iterator.__iter__ "iterator.__iter__") method that returns the iterator object itself so every iterator is also iterable and may be used in most places where other iterables are accepted. One notable exception is code which attempts multiple iteration passes. A container object (such as a [`list`](https://docs.python.org/3/library/stdtypes.html#list "list")) produces a fresh new iterator each time you pass it to the [`iter()`](https://docs.python.org/3/library/functions.html#iter "iter") function or use it in a [`for`](https://docs.python.org/3/reference/compound_stmts.html#for) loop. Attempting this with an iterator will just return the same exhausted iterator object used in the previous iteration pass, making it appear like an empty container.

More information can be found in [Iterator Types](https://docs.python.org/3/library/stdtypes.html#typeiter).

**CPython implementation detail:** CPython does not consistently apply the requirement that an iterator define [`__iter__()`](https://docs.python.org/3/library/stdtypes.html#iterator.__iter__ "iterator.__iter__"). And also please note that CPython does not guarantee behavior of iterator operations.

key [¶](#term-key "Link to this term")

A value that identifies an entry in a. See also.

key function [¶](#term-key-function "Link to this term")

A key function or collation function is a callable that returns a value used for sorting or ordering. For example, [`locale.strxfrm()`](https://docs.python.org/3/library/locale.html#locale.strxfrm "locale.strxfrm") is used to produce a sort key that is aware of locale specific sort conventions.

A number of tools in Python accept key functions to control how elements are ordered or grouped. They include [`min()`](https://docs.python.org/3/library/functions.html#min "min"), [`max()`](https://docs.python.org/3/library/functions.html#max "max"), [`sorted()`](https://docs.python.org/3/library/functions.html#sorted "sorted"), [`list.sort()`](https://docs.python.org/3/library/stdtypes.html#list.sort "list.sort"), [`heapq.merge()`](https://docs.python.org/3/library/heapq.html#heapq.merge "heapq.merge"), [`heapq.nsmallest()`](https://docs.python.org/3/library/heapq.html#heapq.nsmallest "heapq.nsmallest"), [`heapq.nlargest()`](https://docs.python.org/3/library/heapq.html#heapq.nlargest "heapq.nlargest"), and [`itertools.groupby()`](https://docs.python.org/3/library/itertools.html#itertools.groupby "itertools.groupby").

There are several ways to create a key function. For example. the [`str.casefold()`](https://docs.python.org/3/library/stdtypes.html#str.casefold "str.casefold") method can serve as a key function for case insensitive sorts. Alternatively, a key function can be built from a [`lambda`](https://docs.python.org/3/reference/expressions.html#lambda) expression such as `lambda r: (r[0], r[2])`. Also, [`operator.attrgetter()`](https://docs.python.org/3/library/operator.html#operator.attrgetter "operator.attrgetter"), [`operator.itemgetter()`](https://docs.python.org/3/library/operator.html#operator.itemgetter "operator.itemgetter"), and [`operator.methodcaller()`](https://docs.python.org/3/library/operator.html#operator.methodcaller "operator.methodcaller") are three key function constructors. See the [Sorting HOW TO](https://docs.python.org/3/howto/sorting.html#sortinghowto) for examples of how to create and use key functions.

keyword argument [¶](#term-keyword-argument "Link to this term")

See.

lambda [¶](#term-lambda "Link to this term")

An anonymous inline function consisting of a single which is evaluated when the function is called. The syntax to create a lambda function is `lambda [parameters]: expression`

LBYL [¶](#term-LBYL "Link to this term")

Look before you leap. This coding style explicitly tests for pre-conditions before making calls or lookups. This style contrasts with the approach and is characterized by the presence of many [`if`](https://docs.python.org/3/reference/compound_stmts.html#if) statements.

In a multi-threaded environment, the LBYL approach can risk introducing a between “the looking” and “the leaping”. For example, the code, `if key in mapping: return mapping[key]` can fail if another thread removes *key* from *mapping* after the test, but before the lookup. This issue can be solved with or by using the approach. See also.

lexical analyzer [¶](#term-lexical-analyzer "Link to this term")

Formal name for the *tokenizer*; see.

list [¶](#term-list "Link to this term")

A built-in Python. Despite its name it is more akin to an array in other languages than to a linked list since access to elements is *O* (1).

list comprehension [¶](#term-list-comprehension "Link to this term")

A compact way to process all or part of the elements in a sequence and return a list with the results. `result = ['{:#04x}'.format(x) for x in range(256) if x % 2 == 0]` generates a list of strings containing even hex numbers (0x..) in the range from 0 to 255. The [`if`](https://docs.python.org/3/reference/compound_stmts.html#if) clause is optional. If omitted, all elements in `range(256)` are processed.

lock [¶](#term-lock "Link to this term")

A that allows only one thread at a time to access a shared resource. A thread must acquire a lock before accessing the protected resource and release it afterward. If a thread attempts to acquire a lock that is already held by another thread, it will block until the lock becomes available. Python’s [`threading`](https://docs.python.org/3/library/threading.html#module-threading "threading: Thread-based parallelism.") module provides [`Lock`](https://docs.python.org/3/library/threading.html#threading.Lock "threading.Lock") (a basic lock) and [`RLock`](https://docs.python.org/3/library/threading.html#threading.RLock "threading.RLock") (a lock). Locks are used to prevent and ensure access to shared data. Alternative design patterns to locks exist such as queues, producer/consumer patterns, and thread-local state. See also, and reentrant.

lock-free [¶](#term-lock-free "Link to this term")

An operation that does not acquire any and uses atomic CPU instructions to ensure correctness. Lock-free operations can execute concurrently without blocking each other and cannot be blocked by operations that hold locks. In Python, built-in types like [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict") and [`list`](https://docs.python.org/3/library/stdtypes.html#list "list") provide lock-free read operations, which means other threads may observe intermediate states during multi-step modifications even when those modifications hold the.

loader [¶](#term-loader "Link to this term")

An object that loads a module. It must define the `exec_module()` and `create_module()` methods to implement the [`Loader`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader "importlib.abc.Loader") interface. A loader is typically returned by a. See also:

- [Finders and loaders](https://docs.python.org/3/reference/import.html#finders-and-loaders)
- [`importlib.abc.Loader`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader "importlib.abc.Loader")
- [**PEP 302**](https://peps.python.org/pep-0302/)

locale encoding [¶](#term-locale-encoding "Link to this term")

On Unix, it is the encoding of the LC\_CTYPE locale. It can be set with [`locale.setlocale(locale.LC_CTYPE, new_locale)`](https://docs.python.org/3/library/locale.html#locale.setlocale "locale.setlocale").

On Windows, it is the ANSI code page (ex: `"cp1252"`).

On Android and VxWorks, Python uses `"utf-8"` as the locale encoding.

[`locale.getencoding()`](https://docs.python.org/3/library/locale.html#locale.getencoding "locale.getencoding") can be used to get the locale encoding.

See also the.

magic method [¶](#term-magic-method "Link to this term")

An informal synonym for.

mapping [¶](#term-mapping "Link to this term")

A container object that supports arbitrary key lookups and implements the methods specified in the [`collections.abc.Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping "collections.abc.Mapping") or [`collections.abc.MutableMapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.MutableMapping "collections.abc.MutableMapping") [abstract base classes](https://docs.python.org/3/library/collections.abc.html#collections-abstract-base-classes). Examples include [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict"), [`collections.defaultdict`](https://docs.python.org/3/library/collections.html#collections.defaultdict "collections.defaultdict"), [`collections.OrderedDict`](https://docs.python.org/3/library/collections.html#collections.OrderedDict "collections.OrderedDict") and [`collections.Counter`](https://docs.python.org/3/library/collections.html#collections.Counter "collections.Counter").

meta path finder [¶](#term-meta-path-finder "Link to this term")

A returned by a search of [`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path"). Meta path finders are related to, but different from.

See [`importlib.abc.MetaPathFinder`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder "importlib.abc.MetaPathFinder") for the methods that meta path finders implement.

metaclass [¶](#term-metaclass "Link to this term")

The class of a class. Class definitions create a class name, a class dictionary, and a list of base classes. The metaclass is responsible for taking those three arguments and creating the class. Most object oriented programming languages provide a default implementation. What makes Python special is that it is possible to create custom metaclasses. Most users never need this tool, but when the need arises, metaclasses can provide powerful, elegant solutions. They have been used for logging attribute access, adding thread-safety, tracking object creation, implementing singletons, and many other tasks.

More information can be found in [Metaclasses](https://docs.python.org/3/reference/datamodel.html#metaclasses).

method [¶](#term-method "Link to this term")

A function which is defined inside a class body. If called as an attribute of an instance of that class, the method will get the instance object as its first (which is usually called `self`). See and.

method resolution order [¶](#term-method-resolution-order "Link to this term")

Method Resolution Order is the order in which base classes are searched for a member during lookup. See [The Python 2.3 Method Resolution Order](https://docs.python.org/3/howto/mro.html#python-2-3-mro) for details of the algorithm used by the Python interpreter since the 2.3 release.

module [¶](#term-module "Link to this term")

An object that serves as an organizational unit of Python code. Modules have a namespace containing arbitrary Python objects. Modules are loaded into Python by the process of.

See also.

module spec [¶](#term-module-spec "Link to this term")

A namespace containing the import-related information used to load a module. An instance of [`importlib.machinery.ModuleSpec`](https://docs.python.org/3/library/importlib.html#importlib.machinery.ModuleSpec "importlib.machinery.ModuleSpec").

See also [Module specs](https://docs.python.org/3/reference/import.html#module-specs).

MRO [¶](#term-MRO "Link to this term")

See.

mutable [¶](#term-mutable "Link to this term")

An with state that is allowed to change during the course of the program. In multi-threaded programs, mutable objects that are shared between threads require careful synchronization to avoid. See also,, and.

named tuple [¶](#term-named-tuple "Link to this term")

The term “named tuple” applies to any type or class that inherits from tuple and whose indexable elements are also accessible using named attributes. The type or class may have other features as well.

Several built-in types are named tuples, including the values returned by [`time.localtime()`](https://docs.python.org/3/library/time.html#time.localtime "time.localtime") and [`os.stat()`](https://docs.python.org/3/library/os.html#os.stat "os.stat"). Another example is [`sys.float_info`](https://docs.python.org/3/library/sys.html#sys.float_info "sys.float_info"):

```
>>> sys.float_info[1]                   # indexed access
1024
>>> sys.float_info.max_exp              # named field access
1024
>>> isinstance(sys.float_info, tuple)   # kind of tuple
True
```

Some named tuples are built-in types (such as the above examples). Alternatively, a named tuple can be created from a regular class definition that inherits from [`tuple`](https://docs.python.org/3/library/stdtypes.html#tuple "tuple") and that defines named fields. Such a class can be written by hand, or it can be created by inheriting [`typing.NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple "typing.NamedTuple"), or with the factory function [`collections.namedtuple()`](https://docs.python.org/3/library/collections.html#collections.namedtuple "collections.namedtuple"). The latter techniques also add some extra methods that may not be found in hand-written or built-in named tuples.

namespace [¶](#term-namespace "Link to this term")

The place where a variable is stored. Namespaces are implemented as dictionaries. There are the local, global and built-in namespaces as well as nested namespaces in objects (in methods). Namespaces support modularity by preventing naming conflicts. For instance, the functions [`builtins.open`](https://docs.python.org/3/library/functions.html#open "open") and [`os.open()`](https://docs.python.org/3/library/os.html#os.open "os.open") are distinguished by their namespaces. Namespaces also aid readability and maintainability by making it clear which module implements a function. For instance, writing [`random.seed()`](https://docs.python.org/3/library/random.html#random.seed "random.seed") or [`itertools.islice()`](https://docs.python.org/3/library/itertools.html#itertools.islice "itertools.islice") makes it clear that those functions are implemented by the [`random`](https://docs.python.org/3/library/random.html#module-random "random: Generate pseudo-random numbers with various common distributions.") and [`itertools`](https://docs.python.org/3/library/itertools.html#module-itertools "itertools: Functions creating iterators for efficient looping.") modules, respectively.

namespace package [¶](#term-namespace-package "Link to this term")

A which serves only as a container for subpackages. Namespace packages may have no physical representation, and specifically are not like a because they have no `__init__.py` file.

Namespace packages allow several individually installable packages to have a common parent package. Otherwise, it is recommended to use a.

For more information, see [**PEP 420**](https://peps.python.org/pep-0420/) and [Namespace packages](https://docs.python.org/3/reference/import.html#reference-namespace-package).

See also.

native code [¶](#term-native-code "Link to this term")

Code that is compiled to machine instructions and runs directly on the processor, as opposed to code that is interpreted or runs in a virtual machine. In the context of Python, native code typically refers to C, C++, Rust or Fortran code in that can be called from Python. See also extension module.

nested scope [¶](#term-nested-scope "Link to this term")

The ability to refer to a variable in an enclosing definition. For instance, a function defined inside another function can refer to variables in the outer function. Note that nested scopes by default work only for reference and not for assignment. Local variables both read and write in the innermost scope. Likewise, global variables read and write to the global namespace. The [`nonlocal`](https://docs.python.org/3/reference/simple_stmts.html#nonlocal) allows writing to outer scopes.

new-style class [¶](#term-new-style-class "Link to this term")

Old name for the flavor of classes now used for all class objects. In earlier Python versions, only new-style classes could use Python’s newer, versatile features like [`__slots__`](https://docs.python.org/3/reference/datamodel.html#object.__slots__ "object.__slots__"), descriptors, properties, [`__getattribute__()`](https://docs.python.org/3/reference/datamodel.html#object.__getattribute__ "object.__getattribute__"), class methods, and static methods.

non-deterministic [¶](#term-non-deterministic "Link to this term")

Behavior where the outcome of a program can vary between executions with the same inputs. In multi-threaded programs, non-deterministic behavior often results from where the relative timing or interleaving of threads affects the result. Proper synchronization using and other helps ensure deterministic behavior.

object [¶](#term-object "Link to this term")

Any data with state (attributes or value) and defined behavior (methods). Also the ultimate base class of any.

optimized scope [¶](#term-optimized-scope "Link to this term")

A scope where target local variable names are reliably known to the compiler when the code is compiled, allowing optimization of read and write access to these names. The local namespaces for functions, generators, coroutines, comprehensions, and generator expressions are optimized in this fashion. Note: most interpreter optimizations are applied to all scopes, only those relying on a known set of local and nonlocal variable names are restricted to optimized scopes.

optional module [¶](#term-optional-module "Link to this term")

An that is part of the, but may be absent in some builds of, usually due to missing third-party libraries or because the module is not available for a given platform.

See [Requirements for optional modules](https://docs.python.org/3/using/configure.html#optional-module-requirements) for a list of optional modules that require third-party libraries.

package [¶](#term-package "Link to this term")

A Python which can contain submodules or recursively, subpackages. Technically, a package is a Python module with a `__path__` attribute.

See also and.

parallelism [¶](#term-parallelism "Link to this term")

Executing multiple operations at the same time (e.g. on multiple CPU cores). In Python builds with the, only one thread runs Python bytecode at a time, so taking advantage of multiple CPU cores typically involves multiple processes (e.g. [`multiprocessing`](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing "multiprocessing: Process-based parallelism.")) or native extensions that release the GIL. In Python, multiple Python threads can run Python code simultaneously on different cores.

parameter [¶](#term-parameter "Link to this term")

A named entity in a (or method) definition that specifies an (or in some cases, arguments) that the function can accept. There are five kinds of parameter:

- *positional-or-keyword*: specifies an argument that can be passed either or as a keyword argument. This is the default kind of parameter, for example *foo* and *bar* in the following:
	```
	def func(foo, bar=None): ...
	```
- *positional-only*: specifies an argument that can be supplied only by position. Positional-only parameters can be defined by including a `/` character in the parameter list of the function definition after them, for example *posonly1* and *posonly2* in the following:
	```
	def func(posonly1, posonly2, /, positional_or_keyword): ...
	```
- *keyword-only*: specifies an argument that can be supplied only by keyword. Keyword-only parameters can be defined by including a single var-positional parameter or bare `*` in the parameter list of the function definition before them, for example *kw\_only1* and *kw\_only2* in the following:
	```
	def func(arg, *, kw_only1, kw_only2): ...
	```
- *var-positional*: specifies that an arbitrary sequence of positional arguments can be provided (in addition to any positional arguments already accepted by other parameters). Such a parameter can be defined by prepending the parameter name with `*`, for example *args* in the following:
	```
	def func(*args, **kwargs): ...
	```
- *var-keyword*: specifies that arbitrarily many keyword arguments can be provided (in addition to any keyword arguments already accepted by other parameters). Such a parameter can be defined by prepending the parameter name with `**`, for example *kwargs* in the example above.

Parameters can specify both optional and required arguments, as well as default values for some optional arguments.

See also the glossary entry, the FAQ question on [the difference between arguments and parameters](https://docs.python.org/3/faq/programming.html#faq-argument-vs-parameter), the [`inspect.Parameter`](https://docs.python.org/3/library/inspect.html#inspect.Parameter "inspect.Parameter") class, the [Function definitions](https://docs.python.org/3/reference/compound_stmts.html#function) section, and [**PEP 362**](https://peps.python.org/pep-0362/).

per-object lock [¶](#term-per-object-lock "Link to this term")

A associated with an individual object instance rather than a global lock shared across all objects. In Python, built-in types like [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict") and [`list`](https://docs.python.org/3/library/stdtypes.html#list "list") use per-object locks to allow concurrent operations on different objects while serializing operations on the same object. Operations that hold the per-object lock prevent other locking operations on the same object from proceeding, but do not block operations.

path entry [¶](#term-path-entry "Link to this term")

A single location on the which the consults to find modules for importing.

path entry finder [¶](#term-path-entry-finder "Link to this term")

A returned by a callable on [`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks") (i.e. a ) which knows how to locate modules given a.

See [`importlib.abc.PathEntryFinder`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder "importlib.abc.PathEntryFinder") for the methods that path entry finders implement.

path entry hook [¶](#term-path-entry-hook "Link to this term")

A callable on the [`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks") list which returns a if it knows how to find modules on a specific.

path based finder [¶](#term-path-based-finder "Link to this term")

One of the default which searches an for modules.

path-like object [¶](#term-path-like-object "Link to this term")

An object representing a file system path. A path-like object is either a [`str`](https://docs.python.org/3/library/stdtypes.html#str "str") or [`bytes`](https://docs.python.org/3/library/stdtypes.html#bytes "bytes") object representing a path, or an object implementing the [`os.PathLike`](https://docs.python.org/3/library/os.html#os.PathLike "os.PathLike") protocol. An object that supports the `os.PathLike` protocol can be converted to a `str` or `bytes` file system path by calling the [`os.fspath()`](https://docs.python.org/3/library/os.html#os.fspath "os.fspath") function; [`os.fsdecode()`](https://docs.python.org/3/library/os.html#os.fsdecode "os.fsdecode") and [`os.fsencode()`](https://docs.python.org/3/library/os.html#os.fsencode "os.fsencode") can be used to guarantee a `str` or `bytes` result instead, respectively. Introduced by [**PEP 519**](https://peps.python.org/pep-0519/).

PEP [¶](#term-PEP "Link to this term")

Python Enhancement Proposal. A PEP is a design document providing information to the Python community, or describing a new feature for Python or its processes or environment. PEPs should provide a concise technical specification and a rationale for proposed features.

PEPs are intended to be the primary mechanisms for proposing major new features, for collecting community input on an issue, and for documenting the design decisions that have gone into Python. The PEP author is responsible for building consensus within the community and documenting dissenting opinions.

See [**PEP 1**](https://peps.python.org/pep-0001/).

portion [¶](#term-portion "Link to this term")

A set of files in a single directory (possibly stored in a zip file) that contribute to a namespace package, as defined in [**PEP 420**](https://peps.python.org/pep-0420/).

positional argument [¶](#term-positional-argument "Link to this term")

See.

provisional API [¶](#term-provisional-API "Link to this term")

A provisional API is one which has been deliberately excluded from the standard library’s backwards compatibility guarantees. While major changes to such interfaces are not expected, as long as they are marked provisional, backwards incompatible changes (up to and including removal of the interface) may occur if deemed necessary by core developers. Such changes will not be made gratuitously – they will occur only if serious fundamental flaws are uncovered that were missed prior to the inclusion of the API.

Even for provisional APIs, backwards incompatible changes are seen as a “solution of last resort” - every attempt will still be made to find a backwards compatible resolution to any identified problems.

This process allows the standard library to continue to evolve over time, without locking in problematic design errors for extended periods of time. See [**PEP 411**](https://peps.python.org/pep-0411/) for more details.

provisional package [¶](#term-provisional-package "Link to this term")

See.

Python 3000 [¶](#term-Python-3000 "Link to this term")

Nickname for the Python 3.x release line (coined long ago when the release of version 3 was something in the distant future.) This is also abbreviated “Py3k”.

Pythonic [¶](#term-Pythonic "Link to this term")

An idea or piece of code which closely follows the most common idioms of the Python language, rather than implementing code using concepts common to other languages. For example, a common idiom in Python is to loop over all elements of an iterable using a [`for`](https://docs.python.org/3/reference/compound_stmts.html#for) statement. Many other languages don’t have this type of construct, so people unfamiliar with Python sometimes use a numerical counter instead:

```
for i in range(len(food)):
    print(food[i])
```

As opposed to the cleaner, Pythonic method:

```
for piece in food:
    print(piece)
```

qualified name [¶](#term-qualified-name "Link to this term")

A dotted name showing the “path” from a module’s global scope to a class, function or method defined in that module, as defined in [**PEP 3155**](https://peps.python.org/pep-3155/). For top-level functions and classes, the qualified name is the same as the object’s name:

```
>>> class C:
...     class D:
...         def meth(self):
...             pass
...
>>> C.__qualname__
'C'
>>> C.D.__qualname__
'C.D'
>>> C.D.meth.__qualname__
'C.D.meth'
```

When used to refer to modules, the *fully qualified name* means the entire dotted path to the module, including any parent packages, e.g. `email.mime.text`:

```
>>> import email.mime.text
>>> email.mime.text.__name__
'email.mime.text'
```

race condition [¶](#term-race-condition "Link to this term")

A condition of a program where the behavior depends on the relative timing or ordering of events, particularly in multi-threaded programs. Race conditions can lead to behavior and bugs that are difficult to reproduce. A is a specific type of race condition involving unsynchronized access to shared memory. The coding style is particularly susceptible to race conditions in multi-threaded code. Using and other helps prevent race conditions.

reference count [¶](#term-reference-count "Link to this term")

The number of references to an object. When the reference count of an object drops to zero, it is deallocated. Some objects are and have reference counts that are never modified, and therefore the objects are never deallocated. Reference counting is generally not visible to Python code, but it is a key element of the implementation. Programmers can call the [`sys.getrefcount()`](https://docs.python.org/3/library/sys.html#sys.getrefcount "sys.getrefcount") function to return the reference count for a particular object.

In, reference counts are not considered to be stable or well-defined values; the number of references to an object, and how that number is affected by Python code, may be different between versions.

regular package [¶](#term-regular-package "Link to this term")

A traditional, such as a directory containing an `__init__.py` file.

See also.

reentrant [¶](#term-reentrant "Link to this term")

A property of a function or that allows it to be called or acquired multiple times by the same thread without causing errors or a.

For functions, reentrancy means the function can be safely called again before a previous invocation has completed, which is important when functions may be called recursively or from signal handlers. Thread-unsafe functions may be if they’re called reentrantly in a multithreaded program.

For locks, Python’s [`threading.RLock`](https://docs.python.org/3/library/threading.html#threading.RLock "threading.RLock") (reentrant lock) is reentrant, meaning a thread that already holds the lock can acquire it again without blocking. In contrast, [`threading.Lock`](https://docs.python.org/3/library/threading.html#threading.Lock "threading.Lock") is not reentrant - attempting to acquire it twice from the same thread will cause a deadlock.

See also and.

REPL [¶](#term-REPL "Link to this term")

An acronym for the “read–eval–print loop”, another name for the interpreter shell.

\_\_slots\_\_ [¶](#term-__slots__ "Link to this term")

A declaration inside a class that saves memory by pre-declaring space for instance attributes and eliminating instance dictionaries. Though popular, the technique is somewhat tricky to get right and is best reserved for rare cases where there are large numbers of instances in a memory-critical application.

sequence [¶](#term-sequence "Link to this term")

An which supports efficient element access using integer indices via the [`__getitem__()`](https://docs.python.org/3/reference/datamodel.html#object.__getitem__ "object.__getitem__") special method and defines a [`__len__()`](https://docs.python.org/3/reference/datamodel.html#object.__len__ "object.__len__") method that returns the length of the sequence. Some built-in sequence types are [`list`](https://docs.python.org/3/library/stdtypes.html#list "list"), [`str`](https://docs.python.org/3/library/stdtypes.html#str "str"), [`tuple`](https://docs.python.org/3/library/stdtypes.html#tuple "tuple"), and [`bytes`](https://docs.python.org/3/library/stdtypes.html#bytes "bytes"). Note that [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict") also supports `__getitem__()` and `__len__()`, but is considered a mapping rather than a sequence because the lookups use arbitrary keys rather than integers.

The [`collections.abc.Sequence`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Sequence "collections.abc.Sequence") abstract base class defines a much richer interface that goes beyond just [`__getitem__()`](https://docs.python.org/3/reference/datamodel.html#object.__getitem__ "object.__getitem__") and [`__len__()`](https://docs.python.org/3/reference/datamodel.html#object.__len__ "object.__len__"), adding [`count()`](https://docs.python.org/3/library/stdtypes.html#sequence.count "sequence.count"), [`index()`](https://docs.python.org/3/library/stdtypes.html#sequence.index "sequence.index"), [`__contains__()`](https://docs.python.org/3/reference/datamodel.html#object.__contains__ "object.__contains__"), and [`__reversed__()`](https://docs.python.org/3/reference/datamodel.html#object.__reversed__ "object.__reversed__"). Types that implement this expanded interface can be registered explicitly using [`register()`](https://docs.python.org/3/library/abc.html#abc.ABCMeta.register "abc.ABCMeta.register"). For more documentation on sequence methods generally, see [Common Sequence Operations](https://docs.python.org/3/library/stdtypes.html#typesseq-common).

set comprehension [¶](#term-set-comprehension "Link to this term")

A compact way to process all or part of the elements in an iterable and return a set with the results. `results = {c for c in 'abracadabra' if c not in 'abc'}` generates the set of strings `{'r', 'd'}`. See [Displays for lists, sets and dictionaries](https://docs.python.org/3/reference/expressions.html#comprehensions).

single dispatch [¶](#term-single-dispatch "Link to this term")

A form of dispatch where the implementation is chosen based on the type of a single argument.

slice [¶](#term-slice "Link to this term")

An object of type [`slice`](https://docs.python.org/3/library/functions.html#slice "slice"), used to describe a portion of a. A slice object is created when using the [slicing](https://docs.python.org/3/reference/expressions.html#slicings) form of [subscript notation](https://docs.python.org/3/reference/expressions.html#subscriptions), with colons inside square brackets, such as in `variable_name[1:3:5]`.

soft deprecated [¶](#term-soft-deprecated "Link to this term")

A soft deprecated API should not be used in new code, but it is safe for already existing code to use it. The API remains documented and tested, but will not be enhanced further.

Soft deprecation, unlike normal deprecation, does not plan on removing the API and will not emit warnings.

See [PEP 387: Soft Deprecation](https://peps.python.org/pep-0387/#soft-deprecation).

special method [¶](#term-special-method "Link to this term")

A method that is called implicitly by Python to execute a certain operation on a type, such as addition. Such methods have names starting and ending with double underscores. Special methods are documented in [Special method names](https://docs.python.org/3/reference/datamodel.html#specialnames).

standard library [¶](#term-standard-library "Link to this term")

The collection of, and distributed as a part of the official Python interpreter package. The exact membership of the collection may vary based on platform, available system libraries, or other criteria. Documentation can be found at [The Python Standard Library](https://docs.python.org/3/library/index.html#library-index).

See also [`sys.stdlib_module_names`](https://docs.python.org/3/library/sys.html#sys.stdlib_module_names "sys.stdlib_module_names") for a list of all possible standard library module names.

statement [¶](#term-statement "Link to this term")

A statement is part of a suite (a “block” of code). A statement is either an or one of several constructs with a keyword, such as [`if`](https://docs.python.org/3/reference/compound_stmts.html#if), [`while`](https://docs.python.org/3/reference/compound_stmts.html#while) or [`for`](https://docs.python.org/3/reference/compound_stmts.html#for).

static type checker [¶](#term-static-type-checker "Link to this term")

An external tool that reads Python code and analyzes it, looking for issues such as incorrect types. See also and the [`typing`](https://docs.python.org/3/library/typing.html#module-typing "typing: Support for type hints (see :pep:`484`).") module.

stdlib [¶](#term-stdlib "Link to this term")

An abbreviation of.

strong reference [¶](#term-strong-reference "Link to this term")

In Python’s C API, a strong reference is a reference to an object which is owned by the code holding the reference. The strong reference is taken by calling [`Py_INCREF()`](https://docs.python.org/3/c-api/refcounting.html#c.Py_INCREF "Py_INCREF") when the reference is created and released with [`Py_DECREF()`](https://docs.python.org/3/c-api/refcounting.html#c.Py_DECREF "Py_DECREF") when the reference is deleted.

The [`Py_NewRef()`](https://docs.python.org/3/c-api/refcounting.html#c.Py_NewRef "Py_NewRef") function can be used to create a strong reference to an object. Usually, the [`Py_DECREF()`](https://docs.python.org/3/c-api/refcounting.html#c.Py_DECREF "Py_DECREF") function must be called on the strong reference before exiting the scope of the strong reference, to avoid leaking one reference.

See also.

subscript [¶](#term-subscript "Link to this term")

The expression in square brackets of a [subscription expression](https://docs.python.org/3/reference/expressions.html#subscriptions), for example, the `3` in `items[3]`. Usually used to select an element of a container. Also called a when subscripting a, or an when subscripting a.

synchronization primitive [¶](#term-synchronization-primitive "Link to this term")

A basic building block for coordinating (synchronizing) the execution of multiple threads to ensure access to shared resources. Python’s [`threading`](https://docs.python.org/3/library/threading.html#module-threading "threading: Thread-based parallelism.") module provides several synchronization primitives including [`Lock`](https://docs.python.org/3/library/threading.html#threading.Lock "threading.Lock"), [`RLock`](https://docs.python.org/3/library/threading.html#threading.RLock "threading.RLock"), [`Semaphore`](https://docs.python.org/3/library/threading.html#threading.Semaphore "threading.Semaphore"), [`Condition`](https://docs.python.org/3/library/threading.html#threading.Condition "threading.Condition"), [`Event`](https://docs.python.org/3/library/threading.html#threading.Event "threading.Event"), and [`Barrier`](https://docs.python.org/3/library/threading.html#threading.Barrier "threading.Barrier"). Additionally, the [`queue`](https://docs.python.org/3/library/queue.html#module-queue "queue: A synchronized queue class.") module provides multi-producer, multi-consumer queues that are especially useful in multithreaded programs. These primitives help prevent and coordinate thread execution. See also.

t-string [¶](#term-t-string "Link to this term")

t-strings [¶](#term-t-strings "Link to this term")

String literals prefixed with `t` or `T` are commonly called “t-strings” which is short for [template string literals](https://docs.python.org/3/reference/lexical_analysis.html#t-strings).

text encoding [¶](#term-text-encoding "Link to this term")

A string in Python is a sequence of Unicode code points (in range `U+0000` – `U+10FFFF`). To store or transfer a string, it needs to be serialized as a sequence of bytes.

Serializing a string into a sequence of bytes is known as “encoding”, and recreating the string from the sequence of bytes is known as “decoding”.

There are a variety of different text serialization [codecs](https://docs.python.org/3/library/codecs.html#standard-encodings), which are collectively referred to as “text encodings”.

text file [¶](#term-text-file "Link to this term")

A able to read and write [`str`](https://docs.python.org/3/library/stdtypes.html#str "str") objects. Often, a text file actually accesses a byte-oriented datastream and handles the automatically. Examples of text files are files opened in text mode (`'r'` or `'w'`), [`sys.stdin`](https://docs.python.org/3/library/sys.html#sys.stdin "sys.stdin"), [`sys.stdout`](https://docs.python.org/3/library/sys.html#sys.stdout "sys.stdout"), and instances of [`io.StringIO`](https://docs.python.org/3/library/io.html#io.StringIO "io.StringIO").

See also for a file object able to read and write.

thread state [¶](#term-thread-state "Link to this term")

The information used by the runtime to run in an OS thread. For example, this includes the current exception, if any, and the state of the bytecode interpreter.

Each thread state is bound to a single OS thread, but threads may have many thread states available. At most, one of them may be at once.

An is required to call most of Python’s C API, unless a function explicitly documents otherwise. The bytecode interpreter only runs under an attached thread state.

Each thread state belongs to a single interpreter, but each interpreter may have many thread states, including multiple for the same OS thread. Thread states from multiple interpreters may be bound to the same thread, but only one can be in that thread at any given moment.

See [Thread State and the Global Interpreter Lock](https://docs.python.org/3/c-api/threads.html#threads) for more information.

thread-safe [¶](#term-thread-safe "Link to this term")

A module, function, or class that behaves correctly when used by multiple threads concurrently. Thread-safe code uses appropriate like to protect shared mutable state, or is designed to avoid shared mutable state entirely. In the build, built-in types like [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "dict"), [`list`](https://docs.python.org/3/library/stdtypes.html#list "list"), and [`set`](https://docs.python.org/3/library/stdtypes.html#set "set") use internal locking to make many operations thread-safe, although thread safety is not necessarily guaranteed. Code that is not thread-safe may experience and when used in multi-threaded programs.

token [¶](#term-token "Link to this term")

A small unit of source code, generated by the [lexical analyzer](https://docs.python.org/3/reference/lexical_analysis.html#lexical) (also called the *tokenizer*). Names, numbers, strings, operators, newlines and similar are represented by tokens.

The [`tokenize`](https://docs.python.org/3/library/tokenize.html#module-tokenize "tokenize: Lexical scanner for Python source code.") module exposes Python’s lexical analyzer. The [`token`](https://docs.python.org/3/library/token.html#module-token "token: Constants representing terminal nodes of the parse tree.") module contains information on the various types of tokens.

triple-quoted string [¶](#term-triple-quoted-string "Link to this term")

A string which is bound by three instances of either a quotation mark (”) or an apostrophe (‘). While they don’t provide any functionality not available with single-quoted strings, they are useful for a number of reasons. They allow you to include unescaped single and double quotes within a string and they can span multiple lines without the use of the continuation character, making them especially useful when writing docstrings.

type [¶](#term-type "Link to this term")

The type of a Python object determines what kind of object it is; every object has a type. An object’s type is accessible as its [`__class__`](https://docs.python.org/3/reference/datamodel.html#object.__class__ "object.__class__") attribute or can be retrieved with `type(obj)`.

type alias [¶](#term-type-alias "Link to this term")

A synonym for a type, created by assigning the type to an identifier.

Type aliases are useful for simplifying. For example:

```
def remove_gray_shades(
        colors: list[tuple[int, int, int]]) -> list[tuple[int, int, int]]:
    pass
```

could be made more readable like this:

```
Color = tuple[int, int, int]

def remove_gray_shades(colors: list[Color]) -> list[Color]:
    pass
```

See [`typing`](https://docs.python.org/3/library/typing.html#module-typing "typing: Support for type hints (see :pep:`484`).") and [**PEP 484**](https://peps.python.org/pep-0484/), which describe this functionality.

type hint [¶](#term-type-hint "Link to this term")

An that specifies the expected type for a variable, a class attribute, or a function parameter or return value.

Type hints are optional and are not enforced by Python but they are useful to. They can also aid IDEs with code completion and refactoring.

Type hints of global variables, class attributes, and functions, but not local variables, can be accessed using [`typing.get_type_hints()`](https://docs.python.org/3/library/typing.html#typing.get_type_hints "typing.get_type_hints").

See [`typing`](https://docs.python.org/3/library/typing.html#module-typing "typing: Support for type hints (see :pep:`484`).") and [**PEP 484**](https://peps.python.org/pep-0484/), which describe this functionality.

universal newlines [¶](#term-universal-newlines "Link to this term")

A manner of interpreting text streams in which all of the following are recognized as ending a line: the Unix end-of-line convention `'\n'`, the Windows convention `'\r\n'`, and the old Macintosh convention `'\r'`. See [**PEP 278**](https://peps.python.org/pep-0278/) and [**PEP 3116**](https://peps.python.org/pep-3116/), as well as [`bytes.splitlines()`](https://docs.python.org/3/library/stdtypes.html#bytes.splitlines "bytes.splitlines") for an additional use.

variable annotation [¶](#term-variable-annotation "Link to this term")

An of a variable or a class attribute.

When annotating a variable or a class attribute, assignment is optional:

```
class C:
    field: 'annotation'
```

Variable annotations are usually used for: for example this variable is expected to take [`int`](https://docs.python.org/3/library/functions.html#int "int") values:

```
count: int = 0
```

Variable annotation syntax is explained in section [Annotated assignment statements](https://docs.python.org/3/reference/simple_stmts.html#annassign).

See, [**PEP 484**](https://peps.python.org/pep-0484/) and [**PEP 526**](https://peps.python.org/pep-0526/), which describe this functionality. Also see [Annotations Best Practices](https://docs.python.org/3/howto/annotations.html#annotations-howto) for best practices on working with annotations.

virtual environment [¶](#term-virtual-environment "Link to this term")

A cooperatively isolated runtime environment that allows Python users and applications to install and upgrade Python distribution packages without interfering with the behaviour of other Python applications running on the same system.

See also [`venv`](https://docs.python.org/3/library/venv.html#module-venv "venv: Creation of virtual environments.").

virtual machine [¶](#term-virtual-machine "Link to this term")

A computer defined entirely in software. Python’s virtual machine executes the emitted by the bytecode compiler.

walrus operator [¶](#term-walrus-operator "Link to this term")

A light-hearted way to refer to the [assignment expression](https://docs.python.org/3/reference/expressions.html#assignment-expressions) operator `:=` because it looks a bit like a walrus if you turn your head.

Zen of Python [¶](#term-Zen-of-Python "Link to this term")

Listing of Python design principles and philosophies that are helpful in understanding and using the language. The listing can be found by typing “ `import this` ” at the interactive prompt.