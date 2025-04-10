## Basics
1. Node.js is single-threaded in terms of JavaScript execution, but it uses multiple threads under the hood for I/O operations and worker threads."


## Event loop and its shenanigans

1. The event loop runs on the main thread, where all JavaScript executes.
2. It cycles through multiple phases, each with its own queue of callbacks (e.g. timers, I/O, etc.).
3. Each phase handles a specific type of task, like timers, I/O callbacks, or microtasks.
4. When a phase is active, the event loop processes callbacks in that phase’s queue, one by one.
5. Once done (or the queue is paused/yielded), it moves to the next phase and repeats.
6. I/O-bound tasks (like fs.readFile) are offloaded to the libuv thread pool, which handles them in the background and queues their callbacks back into the event loop when finished.
7. For CPU-bound custom tasks, use the worker_threads module — these spawn separate native threads, not part of the libuv thread pool, and communicate via message passing (postMessage / 'message').
8. async or promise callbacks are part of microtask queue whose all tasks are completed at once before moving on.
