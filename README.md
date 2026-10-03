# JavaScript, Python, and Java: How They Work

This guide compares JavaScript (including Node.js), Python (especially CPython), and Java. It focuses on variables, objects, memory, functions, concurrency, and what happens while a program runs.

## 15-Point Comparison: Node.js, Python, and Java

Node.js is a runtime for JavaScript, not a separate programming language. The table compares Node.js as an application platform with the commonly used CPython runtime and Java on the JVM.

| # | Topic | Node.js (JavaScript) | Python (CPython) | Java (JVM) |
| --- | --- | --- | --- | --- |
| 1 | Language and platform | JavaScript language running in Node.js; Node adds server-side APIs. | Python language running here on CPython. | Java language compiled for the JVM. |
| 2 | Typing | Dynamically typed; values have runtime types. | Dynamically typed; values have runtime types. | Statically typed; most type compatibility is checked by the compiler. |
| 3 | Variable declarations | `const` and `let` are block-scoped; `var` is function-scoped. | Names are bound by assignment; no declaration keyword is required. | Local variables declare a type, such as `int` or `String`. |
| 4 | Common value model | Primitive values plus objects; arrays and functions are objects. | All values, including numbers and functions, are objects. | Primitive values plus references to objects. |
| 5 | Assignment | Object references are copied; aliases can mutate the same object. | Names become bound to objects; aliases can mutate the same object. | Primitive values are copied; object reference values are copied, so aliases can mutate the same object. |
| 6 | Function model | First-class functions and closures are built into the language. | First-class function objects and closures are built into the language. | Methods are class members; lambdas and method references target functional interfaces. |
| 7 | Typical execution | V8 parses, compiles, and JIT-optimizes JavaScript as it runs. | CPython compiles source to bytecode and executes it in its virtual machine. | Java source compiles to bytecode; the JVM interprets and may JIT-compile hot code. |
| 8 | Memory management | V8 uses tracing garbage collection; details vary by version. | CPython primarily uses reference counting plus cyclic garbage collection. | JVM uses tracing garbage collectors, selectable/configurable for the application. |
| 9 | Main concurrency model | Event loop with asynchronous I/O; worker threads or processes for CPU parallelism. | `asyncio`, threads, or processes; conventional CPython's GIL affects Python-bytecode parallelism. | Threads and concurrency APIs; virtual threads and asynchronous APIs are also available. |
| 10 | I/O-bound work | Well suited to many concurrent I/O operations when handlers avoid blocking the event loop. | `asyncio` or threads can handle concurrent I/O; libraries must support the chosen model. | Threads, virtual threads, or asynchronous APIs can support concurrent I/O. |
| 11 | CPU-bound work | CPU-heavy JavaScript blocks its event loop; use workers, processes, or native code when appropriate. | Use native libraries or processes for CPU parallelism; available options depend on the Python build. | Threads can execute in parallel across cores; algorithm and synchronization costs still matter. |
| 12 | Runtime and system APIs | Node APIs plus native components, including libuv; browser APIs are not automatically available. | Standard library and installed packages provide runtime and system APIs. | JVM libraries and the Java platform provide runtime and system APIs. |
| 13 | Common server frameworks | Express, Fastify, and other Node frameworks. | Django, Flask, FastAPI, and other Python frameworks. | Spring, Jakarta EE, and other Java frameworks. |
| 14 | Common strengths | Network services, APIs, and applications sharing JavaScript across server and browser code. | Automation, scripting, data work, and web services with a broad package ecosystem. | Large services, long-running applications, and systems that benefit from JVM tooling and static typing. |
| 15 | Performance decision | Depends on workload, libraries, I/O, allocation, and V8 optimization; benchmark the real service. | Depends on workload, native extensions, libraries, and runtime; benchmark representative code. | Depends on workload, JVM warmup/JIT, libraries, memory, and configuration; benchmark after warmup where relevant. |

## 1. Declaring Variables and Typing

| Language | Typical declaration | What the declaration means |
| --- | --- | --- |
| JavaScript | `let count = 1;`, `const name = "Ada";`, `var oldStyle = true;` | `let` and `const` are block-scoped. `const` prevents rebinding the name; it does not make an object immutable. `var` is function-scoped (or global-scoped) and has older hoisting behavior. Prefer `const` by default and `let` when rebinding is needed. |
| Python | `count = 1` | Assignment binds a name to an object. There is no keyword for declaring a variable, and a name can later be bound to an object of a different type. |
| Java | `int count = 1;`, `String name = "Ada";` | A local variable has a declared type. The compiler checks that uses and assignments are compatible with that type. |

### Static and dynamic typing

Java is statically typed: types are checked primarily by the compiler before the program runs. JavaScript and Python are dynamically typed: the type of a value is determined at runtime, and an operation can fail at runtime if the values do not support it.

This is about when and how types are checked, not whether a language has types. JavaScript and Python both have types. Also, “compile time versus runtime” is only a shorthand: JavaScript engines and Python implementations may compile code internally, and Java programs still perform runtime checks such as bounds checks and casts.

```javascript
let value = 10;
value = "ten"; // Allowed: the binding can refer to a different kind of value.
```

```python
value = 10
value = "ten"  # Also allowed.
```

```java
int value = 10;
// value = "ten"; // Compile-time error: String is not assignable to int.
```

## 2. Primitive Values, Objects, and References

### JavaScript

JavaScript has primitive values: `undefined`, `null`, booleans, numbers, bigints, strings, and symbols. Objects include ordinary objects, arrays, and functions. A variable or property can hold a primitive value or a reference to an object.

### Python

In Python, values are objects, including integers, strings, lists, and functions. A variable is a name bound to an object. Some objects are immutable (such as strings, integers, and tuples); others are mutable (such as lists and dictionaries).

### Java

Java distinguishes primitive types such as `int`, `double`, and `boolean` from reference types such as `String`, arrays, and class instances. A primitive variable contains its primitive value. A reference variable contains a reference to an object, or `null`.

The exact physical representation is implementation-dependent. The useful model is about language behavior: whether a value is mutable, whether a binding can be changed, and whether multiple variables can refer to the same object.

## 3. Assignment Is Not Usually a Deep Copy

Assignment gives a variable a value. When that value refers to a mutable object, assigning it to another variable usually copies the reference, not the object. Both names then provide access to the same object.

### JavaScript example

```javascript
const first = { score: 1 };
const second = first;
second.score = 2;
console.log(first.score); // 2: both names refer to the same object.
```

`const` prevents `first` from being rebound, but does not prevent changing `first.score`.

### Python example

```python
first = [1, 2]
second = first
second.append(3)
print(first)  # [1, 2, 3]
```

### Java example

```java
int[] first = {1, 2};
int[] second = first;
second[0] = 9;
System.out.println(first[0]); // 9: both references identify the same array.
```

For a shallow copy, use an appropriate operation for the type, such as JavaScript `[...items]`, Python `items.copy()`, or Java `array.clone()`. A shallow copy duplicates the outer container, but nested objects may still be shared. A deep copy recursively duplicates nested data, where supported and appropriate.

Strings are immutable in all three languages. An operation that appears to change a string creates or produces another string value; it does not mutate the existing string object.

## 4. Mutable and Immutable Values

| Language | Common immutable examples | Common mutable examples |
| --- | --- | --- |
| JavaScript | strings, numbers, booleans, `bigint`, symbols | objects, arrays, `Map`, `Set` |
| Python | integers, floats, strings, tuples (their contained objects may still be mutable) | lists, dictionaries, sets, most class instances |
| Java | primitives, `String`, many value-like library types | arrays, most collection implementations, most ordinary class instances |

Immutability belongs to a value or object, not necessarily to a variable. A variable may be rebound even if its old value was immutable. Conversely, a constant binding can refer to a mutable object.

## 5. Stack, Heap, and Memory Models

The stack and heap are useful concepts, but “variables live on the stack and objects live on the heap” is not a complete or universally accurate rule.

- A **call stack** tracks active function or method calls. A frame commonly includes return information and storage for parameters and local state.
- A **heap** is memory used for dynamically managed data, including many objects and collections.
- Implementations may optimize or represent values differently. A compiler or JIT can keep values in registers, eliminate allocations, or move data. The language specification usually does not promise a particular stack-versus-heap layout.
- In Java, local primitive values and references are often described as being in a stack frame, while objects are often described as being on the heap. This is a helpful teaching model, not a guarantee for every optimized execution.
- In Python and JavaScript, local names are associated with runtime-managed values; the implementation controls their physical representation. Do not infer object lifetime solely from a source-level variable's presumed stack location.

The stack is primarily about nested execution and call lifetime. The heap is primarily about dynamically managed data and object lifetime. Garbage collection, optimization, closures, and runtime implementation make the simple slogan incomplete.

## 6. Function Calls and Their State

When a function or method is called, the runtime establishes state for that call. Conceptually, that state includes:

- **Parameters**, which receive the argument values.
- **Local variables**, which are available to that call.
- **A return point**, so execution can continue after the call.
- **A return value**, if the function produces one.

Calls are nested on the call stack: a caller pauses while a callee runs, then resumes when the callee returns. Recursion creates additional active calls. Implementations may optimize calls, so this is a semantic model rather than a promise about physical layout.

## 7. Functions as Values

- **JavaScript:** functions are first-class values. They can be stored, passed as arguments, returned, and created inline. Node.js uses the same JavaScript function model.
- **Python:** functions are first-class objects and can likewise be stored, passed, and returned.
- **Java:** methods belong to classes or objects and are not standalone values. Lambdas and method references can be used where a functional interface is expected, such as `Runnable` or `Function<T, R>`.

## 8. Argument Passing: What “By Value” Means

Java and JavaScript pass argument values by value. Python is commonly described as using **call by sharing** (also called object-sharing): the parameter is bound to the same object supplied by the caller. These descriptions lead to a practical point shared by all three: rebinding a parameter does not rebind the caller's variable, but mutating a shared mutable object can be visible to the caller.

### Mutating an object passed to a function

```javascript
function addItem(items) {
	items.push("new");
}
const values = ["old"];
addItem(values);
console.log(values); // ["old", "new"]
```

```python
def add_item(items):
		items.append("new")

values = ["old"]
add_item(values)
print(values)  # ['old', 'new']
```

```java
static void addItem(java.util.List<String> items) {
		items.add("new");
}
```

In Java, the reference value is copied into the parameter. The parameter and caller's variable refer to the same list, so mutation is visible. Assigning a new list to `items` inside the method would not change the caller's variable.

## 9. Closures and Captured Variables

A **closure** is a function together with access to variables from its surrounding lexical scope. It allows a function to use those variables even after the outer function has returned. The runtime keeps the captured state alive as long as it is needed and reachable.

### JavaScript

```javascript
function makeCounter() {
	let count = 0;
	return () => ++count;
}

const next = makeCounter();
next(); // 1
next(); // 2
```

The returned function keeps access to `count` after `makeCounter` has returned.

### Python

```python
def make_counter():
		count = 0

		def next_count():
				nonlocal count
				count += 1
				return count

		return next_count
```

The nested function retains access to the enclosing binding. `nonlocal` allows rebinding that enclosing local name.

### Java

Java lambdas can capture local variables that are final or **effectively final** (assigned once and not subsequently changed). A captured local's value remains available to the lambda after the method returns. Java does not allow a lambda to reassign that captured local. A captured object reference can still refer to a mutable object, and that object's state may be changed.

Captured state outlives the original call because the function or lambda retains access to it. That can also keep larger object graphs alive, so closures are worth considering when retaining callbacks for a long time.

## 10. Garbage Collection and Object Lifetime

Garbage collection reclaims memory for objects that the runtime can prove are no longer reachable (or otherwise no longer needed under its rules). It relieves programs from manually freeing every object, but it does not guarantee immediate reclamation at the moment an object becomes unused.

An object generally becomes **eligible for collection** when no live program path can reach it. Eligibility is not the same as immediate collection or immediate return of memory to the operating system.

- `delete` in JavaScript removes a configurable object property; it does not directly free the object. The object may remain reachable through another reference.
- Python's `del name` removes a name binding; it does not directly free the object. Other references may remain, and reclamation timing depends on the implementation.
- Java has no general-purpose `delete` for objects. Assigning `null` or leaving a scope can remove a reference, but does not command immediate collection.

### Broad collector comparison

- **JavaScript engines such as V8:** tracing garbage collection identifies reachable objects from roots and reclaims unreachable ones. Exact algorithms vary by engine and version.
- **CPython:** primarily uses reference counting, with a cyclic garbage collector to find certain unreachable reference cycles. Reference counting can reclaim many objects promptly, but cycles and implementation details mean `del` is not a universal immediate-release command.
- **Java on the JVM:** uses tracing garbage collectors. The selected collector and collection timing depend on JVM configuration and runtime conditions.

Garbage collection manages memory; it does not determine whether an object is still useful to the application. A **memory leak despite GC** occurs when the program unintentionally keeps unused objects reachable. Common causes include unbounded caches, global references, event listeners that are never removed, and long-lived collections that keep accumulating entries.

## 11. Runtimes, Compilation, and Interpretation

The language is not the same thing as its runtime. A runtime supplies execution services such as memory management, libraries, scheduling, and integration with the operating system.

- **JavaScript in Node.js:** Node.js combines the V8 JavaScript engine with Node APIs and native components, including libuv for event-loop and asynchronous I/O support. Browsers use JavaScript engines too, but provide different host APIs.
- **Python:** CPython is the most widely used implementation, but Python also has other implementations. CPython commonly compiles source to bytecode and executes it in its virtual machine.
- **Java:** Java source is normally compiled to JVM bytecode. A Java Virtual Machine loads and verifies classes, executes bytecode, and may use a JIT compiler to compile frequently executed code to machine code.

“Compiled versus interpreted” is too simplistic. A runtime may combine parsing, bytecode generation, interpretation, JIT compilation, and optimization. JavaScript engines may compile and optimize code at runtime; CPython uses bytecode; and the JVM may interpret some code while compiling hot paths. These strategies do not by themselves determine performance.

## 12. Event Loops, Threads, and Workloads

- **Node.js:** commonly runs JavaScript callbacks on an event loop. Asynchronous I/O lets the process handle many waiting operations without dedicating one JavaScript thread to each request. CPU-heavy synchronous work can still block that event loop. Worker threads or processes are options for parallel CPU work.
- **Python:** `asyncio` provides event-loop-based concurrency, especially useful for I/O-bound tasks. Threads can help with blocking I/O. In conventional CPython builds, the Global Interpreter Lock (GIL) limits simultaneous execution of Python bytecode by threads, though native extensions may release it and free-threaded builds are also available. Multiprocessing is another option for CPU parallelism.
- **Java:** supports threads and a broad concurrency toolkit. Threads can run tasks concurrently; actual parallel execution depends on available cores and the work. Asynchronous and virtual-thread approaches can also help with large numbers of I/O-bound tasks.

**I/O-bound** work spends much of its time waiting for a network, disk, or database. **CPU-bound** work spends most of its time computing. Event loops are often effective for many concurrent I/O waits; CPU-heavy work needs parallel execution or more compute capacity to reduce elapsed time.

## 13. A Request Through an Application

A simplified HTTP request might move through these steps:

1. A server receives the request and dispatches it to a handler.
2. The handler calls functions or methods, creating local execution state and binding parameters.
3. The code creates or updates objects and values while validating input and applying business rules.
4. It may await or perform database/API I/O. The runtime and framework manage the waiting and resumption behavior.
5. The handler builds a result, and the server serializes it into an HTTP response.
6. Temporary objects become collectible only when no longer reachable; shared caches or retained callbacks may keep objects alive longer.

The exact details depend on the web framework, runtime, database driver, and whether the code is synchronous or asynchronous.

## 14. What Happens for `a + b`?

The expression looks similar in all three languages, but type rules and runtime work differ.

### Python

```python
result = a + b
```

Python looks up the objects bound to `a` and `b`, then applies the addition protocol (typically a type's `__add__`, with fallback behavior where applicable). The operation may add numbers, concatenate sequences, or fail with `TypeError`, depending on the operand types. The result object is then bound to `result`; this does not necessarily copy either operand.

### JavaScript

```javascript
const result = a + b;
```

JavaScript evaluates `a` and `b`, applies its primitive-conversion and `+` rules, then binds the result to a new constant binding named `result`. For numbers, `+` adds; if string conversion leads to string concatenation, it concatenates. For example, `1 + 2` is `3`, while `"1" + 2` is `"12"`. `const` means the binding cannot be reassigned, not that an object result would be immutable.

### Java

```java
int result = a + b;
```

The compiler resolves the operand types and the applicable `+` operation. For `int` operands it performs integer addition, subject to Java's integer overflow behavior, then stores the value in the local variable `result`. If operands are strings, `+` performs string concatenation instead. Incompatible operand types generally produce a compile-time error.

In all cases, a real execution may also involve optimized machine instructions, temporary values, or allocation decisions made by the runtime. The source line alone does not reveal the exact physical memory operations.

## 15. Performance: Ask About the Workload

There is no useful universal answer to “which language is fastest?” Performance depends on the workload and the whole system:

- Is the task CPU-bound or I/O-bound?
- How much data is allocated, retained, or copied?
- What are the garbage-collection and memory-pressure effects?
- What algorithms, libraries, database queries, and network calls dominate?
- How does the runtime optimize this specific code, and what concurrency model fits the workload?

For real applications, measure representative workloads and profile before choosing an optimization. Architecture and algorithm choices often matter more than a language-level slogan.

## 16. Three Lifetimes to Keep Separate

- **Variable/name lifetime:** how long a binding can be accessed in its scope or captured environment. Scope ending does not necessarily mean its object is immediately reclaimed.
- **Object reachability:** whether a live reference path still leads to the object. Several variables can refer to the same object.
- **Collection and memory release:** when the runtime reclaims an unreachable object, and when reclaimed memory is reused or returned to the operating system. These events are not necessarily simultaneous.

Keeping these three ideas separate makes assignment, closures, garbage collection, and memory usage much easier to reason about.