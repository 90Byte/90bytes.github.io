---
title: "Post: [JS] event loop, Task"
last_modified_at: 2021-08-09T01:20:37+0900
categories:
  - Blog
tags:
  - JavaScript
  - event loop
  - asynchronous
  - concurrency
---
# Overview
It's often said that JavaScript is a single-threaded language. However, at one time, it was considered a revolutionary solution to asynchronous I/O, a way to save us from the risk of deadlock caused by numerous threads. Let's delve into this seemingly paradoxical proposition and understand the structure of asynchronous tasks in JavaScript.

# Concurrency !== Threads
First, what are threads? Abstractly, a thread refers to a single sequence of work. In programming, we often know the concept of call stacks. When a function is called, it stacks on the call stack, and if another function is called inside, it stacks on top again. Our program ends when the call stack is ultimately popped; it returns the main's return value from the program's entry point to the operating system.

Technically, a single sequence of work means executing functions sequentially using a call stack. Multi-threading means having multiple sequences of work and switching between these sequences rapidly to perform tasks. With a typical approach similar to other languages, one might compulsively feel that concurrent tasks should be placed on threads.

However, using threads is just one tool to implement concurrent tasks.

# So, what about JavaScript?
As mentioned, JavaScript is said to be single-threaded. Then how does JavaScript handle concurrent tasks?
Let's go back to the origins of JavaScript. JavaScript was initially created for controlling web frontends within browsers. Therefore, JavaScript needed a solution to fundamentally prevent deadlocks that could severely hinder user experience. The solution was to make multithreading impossible for developers.

So, how does JavaScript operate? It pulls tasks from a task queue called the event loop and performs them in a single call stack. Once a task on the call stack is completely finished, it fetches the next task from the queue.

Then how about heavy operations like I/O? This is handled by Web API, which functions like libc for JavaScript, processing these operations in threads hidden behind the JavaScript engine, and responding with callbacks. Precisely, it starts a task with a callback by inserting it into the event loop task queue. JavaScript frequently uses patterns like this:
```javascript
setTimeout(function(){
// some task..
}, 0);
```

In reality, this pattern is about placing content you don't want to execute in the current call stack into the event loop task queue using the Web API. It's not about executing concurrent work like threads.

This topic is detailed in [a well-known keynote](https://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html). In essence, JavaScript being single-threaded is from the developer's level perspective, but in reality, it uses multiple threads. However, developers cannot create deadlocks as they would in other languages using the traditional approach. Thanks to these callbacks, closures in JavaScript could become rapidly popular. If a callback can access the environment in which it was created, developers can sequentially understand asynchronous work flows, such as:
```
function A(someVar){
	someAPI(function(someResponse) { // this is callback
		const someResult = someTask(someVar, someResponse);
		doSomething(someResult);
	});
}
```


People new to JavaScript might initially be terrified by this plethora of callbacks and may look for synchronous APIs or resort to busy waiting. Using synchronous APIs can delay rendering tasks, causing user-perceived delays. This is why synchronous APIs are gradually being deprecated. Busy waiting is even more severe; it can lead right into deadlocks.

# More on tasks
However, regardless of using closures, the feast of callbacks can sometimes be overwhelming. For example:
```javascript
function A(callback1) {
	function doSomething(callback2) {
		callback2(function(callback3) {
			callback3(function(callback4) {
				callback4(callback1);
			});
		});
	}
    return doSomething;
}
```

Honestly, it was something I just made up, and even I don't understand what it is supposed to do. Callbacks can get complicated and hard to understand, surpassing objc calls. This is where Promises and async functions come in.

Promises are objects in JavaScript that abstract asynchronous behavior, officially adopting one of the most famous patterns to escape the callback pattern since es6. Here's an example from the Mozilla Foundation document:
```javascript
let myFirstPromise = new Promise((resolve, reject) => {
  // We call resolve(...) when what we were doing asynchronously was successful, and reject(...) when it failed.
  // In this example, we use setTimeout(...) to simulate async code.
  // In reality, you will probably be using something like XHR or an HTML5 API.
  setTimeout(function(){
    resolve("Success!"); // Yay! Everything went well!
  }, 250);
});
myFirstPromise.then((successMessage) => {
  // successMessage is whatever we passed in the resolve(...) function above.
  // It doesn't have to be a string, but if it is only a succeed message, it probably will be.
  console.log("Yay! " + successMessage);
});
```

In actual examples, the sequential flow of asynchronous tasks is created with then and catch.

The async syntax allows expressing a function as asynchronous. Let’s look at a Mozilla Foundation document example:
```javascript
function resolveAfter2Seconds() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('resolved');
    }, 2000);
  });
}
async function asyncCall() {
  console.log('calling');
  const result = await resolveAfter2Seconds();
  console.log(result);
  // expected output: "resolved"
}
asyncCall();
```

Functions prefixed with the async keyword become async functions. The returned value of an async function is considered resolved for a Promise, and errors thrown are catchable like in a try{} catch{} block. Calling an async function in a non-async context returns a Promise object, not the returned value. Within an async function, Promise objects can be treated like functions, and the await keyword allows for waiting for a value, effectively handling rejected errors from Promises. Thus, async functions enable understanding asynchronous tasks performed as Promises in a more natural, synchronous flow.

To operate like the setTimeout example with Promises, you can write as follows:
```javascript
const p = new Promise(function(resolve, reject) {
	resolve();
}).then(function() {
	// do something
});
```

However, strictly speaking, the two behave differently internally. Though earlier the task queue was described as though it was a single queue, in fact, there are multiple queues: the Macro task queue where most tasks enter, the Micro task queue that executes tasks somewhat faster(?), and the rendering queue where rendering tasks are performed.

![event loop](https://javascript.info/article/event-loop/eventLoop-full.svg)
[source](https://javascript.info/event-loop)

The event loop first executes the oldest task in the Macro task queue. Then, it carries out all tasks in the Micro task queue until it is empty and performs rendering tasks if any, before repeating the process. Nevertheless, the order appears as if Macro tasks are performed first, but since it waits after performing one, it actually has the lowest priority. Thus, for tasks that need to be dealt with as a matter of priority, they should be placed in the Micro task queue. While you can directly insert tasks into the Micro task queue using the queueMicrotask() API, resolving a Promise also does the job as Promises are executed in the microtask queue. Meanwhile, setTimeout is considered a macro task.

The keynote mentioned earlier suggests ideally rendering every 16.6 milliseconds, but unless the stack is cleared, rendering won't occur. Therefore, frontend developers must always suspect reasons for rendering delays. Is a task taking too long? Then, it needs to be broken down. Are there too many tasks occurring? Consider debouncing or throttling.

# Real concurrent tasks?
If you've read this far, you'll understand, but if you want to perform heavy tasks not provided by the Web API in JavaScript, it's challenging. Suppose a frontend encryption library needs to generate a large prime number. If a single task handles it all, rendering will face significant delays because rendering occurs only after the task leaves the call stack. Sitting for 3-4 minutes to find a large prime number would leave rendering hanging. One solution is to break the logic into smaller tasks, but for such jobs, there's the Web Worker API.

Web Workers genuinely create threads for separate tasks. What was said about single-threading was not entirely true. Sorry. Web Workers have been usable since es5.

However, there are several limitations and rules for Web Workers:

1. Web Workers require a JavaScript file containing the code to perform within the worker.
2. Web Workers have a different global context from the main thread's window.
3. Web Workers cannot access the DOM for rendering.
4. The main thread and the Web Worker communicate through a messaging system. (In this case, the data doesn't go as a reference but as a copied value.)
5. Workers are real OS-level threads.
6. Workers can also use XHR, etc., within.

In a word, it's an API that developers can use for genuine background execution of user-level logic.

# In conclusion
I wrote about JavaScript's event loop and operational principles to the extent of my limited knowledge. There might be inaccuracies, so please refer to Mozilla Foundation and other references for understanding.

[^1]: Synchronous/asynchronous refers to whether the completion of a task is checked by the caller (synchronous) or indirectly received through callbacks (asynchronous). Blocking/non-blocking refers to whether the called function holds control and makes you wait (blocking) or immediately returns control (non-blocking).
[^2]: Node.js also provides APIs similar to the Web API, but given it operates in a server environment, it's not entirely the same and offers additional functionalities, excludes some features, or provides differently implemented APIs.
[^3]: As a side note, these hidden threads are said to be user-level threads according to [this post](https://stackoverflow.com/questions/16707098/node-js-kernel-mode-threading). There are user-level threads, kernel-level threads, and combined threads. Kernel-level threads are the lowest level, managed by the operating system, and a process can run on a single kernel-level thread. This means in a multi-core environment, kernel-level threads can indeed execute simultaneously. A program has at least one kernel-level thread. User-level threads are implemented at the language or framework level and are abstract threads from the operating system's perspective.
[^4]: The deadlock mentioned here refers to the conventional deadlock that occurs due to lock between threads. JavaScript does not move to the next task until the current call stack is completely finished. Thus, the call stack of JavaScript and Web APIs operate in a non-preemptive, cooperative scheduling relationship. Be aware that JavaScript's concurrency mechanism can be mistakenly understood like preemptive scheduling, common in general threads, and can sufficiently cause [deadlocks](http://webreflection.blogspot.com/2007/06/simplest-javascript-deadlock.html). This is why busy-waiting in callbacks is considered an anti-pattern in JavaScript.
[^5]: Conceptually, a closure (Closure) means a function that shares scope with its creation location. In other words, if function A creates function B, B can remember and access the local variables within A. JavaScript utilizes the lexical environment for this, which I'll discuss in another post when I have the chance.
[^6]: Of course, this doesn't mean actually waiting in a busy-wait manner. It effectively has the same effect as handling with a callback.