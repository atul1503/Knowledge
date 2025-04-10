## Basics
1. Node.js is single-threaded in terms of JavaScript execution, but it uses multiple threads under the hood for I/O operations and worker threads."


## Event loop and its shenanigans

1. The event loop runs on the main thread, where all JavaScript executes.
2. The event loop executes everything in a single thread called the call stack synchronously.
3. One the call stack is empty or all synchronous tasks are done, event loop fetches a callback from microstask queue. It processes it and then fetches another microtask queue and executes it.
4. Once the event loop has fetched and processed all microtask callbacks, it then does the same for task queue callbacks.
5. But microtask queue task has priority over task queue task. So if the call stack is empty and event loop has tasks both in microtask queue and task queue, it fetches and executes microtask from the microtask queue. After its done all the microtasks, it does the task queue.
6. promise callbacks, queueMicrotask(), process.netTick() etc are tasks that are put in microtask queue.
7. set timeout and timer related callbacks, i/o callbacks, setImmediate etc are task put in task queue.
