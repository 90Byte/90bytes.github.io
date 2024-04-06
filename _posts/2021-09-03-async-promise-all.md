---
title: "Post: How to Implement Promise.all in an Async Function"
last_modified_at: 2021-09-03T06:26:35:37+0900
categories:
  - Blog
tags:
  - JavaScript
  - Promise
  - Async
---
In a previous post, I introduced the async function as a keyword that enables the implementation of promises in a synchronous manner. 

However, I would like to amend and reintroduce that there are some performance differences, not entirely identical. Strictly speaking, I consider it more of a common anti-pattern that causes performance differences rather than performance differences themselves.

# What's Different?
The await keyword that can be used within an async function technically stops subsequent statements until the promise object behind the keyword is fulfilled. This means that the trailing statement corresponds to the code in the promise's then.

```javascript
Promise.resolve('Hello').then((greeting) => {
 console.log(greeting);
});
```

For example, using a promise like above

```javascript
async function hello() {
 const greeting = await Promise.resolve('Hello');
 console.log(greeting);
}
```

means that it can be converted to synchronous in this manner.

But let's think about a situation where multiple asynchronous tasks need to be used.

```javascript
async function downloadAll() {
 const image1 = await fetch('image1.png');
 const image2 = await fetch('image2.png');
 const image3 = await fetch('image3.png');
 
 doSomething(image1, image2, image3);
}
```

The intention here might have been to fetch images simultaneously and then call doSomething. 

However, due to the design of the await keyword, in reality, it will operate serially like fetch('image1').then -> fetch('image2').then -> ... doSomething. 

This leads to a logical bottleneck. If Promise were used, it would have been designed to call doSomething after all promises are fulfilled like Promise.all([fetch('image1'), fetch('image2'), fetch('image3')]).then... 

Nevertheless, There is a pattern in async functions to implement this operation.

```javascript
async function timeTest() {
 const timeoutPromise1 = timeoutPromise(3000);
 const timeoutPromise2 = timeoutPromise(3000);
 const timeoutPromise3 = timeoutPromise(3000);

 await timeoutPromise1;
 await timeoutPromise2;
 await timeoutPromise3;
}
```

Defining promise objects simultaneously after await means the following happens.

```javascript
const image1 = await fetch('image1.png');
```

In the code above, fetch is executed first, and then the progress is paused in await until the promise caused by fetch is completed. Once fetch is fulfilled, the resolved value is assigned to const image1. 

Therefore, if you want to implement an operation like promise.all within async, as in the example above, you just need to store all promises in variables and apply await to all of them.