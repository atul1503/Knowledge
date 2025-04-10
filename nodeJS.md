## Basics
1. Node.js is single-threaded in terms of JavaScript execution, but it uses multiple threads under the hood for I/O operations and worker threads.
2. EventEmitter is from "events" pakcage which gives a class EventEmitter whose object can be used to register event(string) and register listeners and then you can emit that event and all its listeners will run. Listeners here are functions actually.
3. you can use `url.parse(your_url)` to parse the url into its domain, query param etc.
4. use `fork()` to start another nodejs process.In that process, use `process.send(message)` to send message to its parent. Parent will use the  child refrence created from `fork` to send message to its child with `child.send(message)`.
5. use `spawn()` to spawn any command line process with args.


## Event loop and its shenanigans

1. The event loop runs on the main thread, where all JavaScript executes.
2. The event loop executes everything in a single thread called the call stack synchronously.
3. One the call stack is empty or all synchronous tasks are done, event loop fetches a callback from microstask queue. It processes it and then fetches another microtask queue and executes it.
4. Once the event loop has fetched and processed all microtask callbacks, it then does the same for task queue callbacks.
5. But microtask queue task has priority over task queue task. So if the call stack is empty and event loop has tasks both in microtask queue and task queue, it fetches and executes microtask from the microtask queue. After its done all the microtasks, it does the task queue.
6. promise callbacks, queueMicrotask(), process.netTick() etc are tasks that are put in microtask queue.
7. set timeout and timer related callbacks, i/o callbacks, setImmediate etc are task put in task queue.
