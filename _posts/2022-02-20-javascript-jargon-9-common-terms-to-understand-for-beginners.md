---
layout: post
title: "Javascript Jargon - 9 Common Terms to Understand for Beginners"
date: 2022-02-20
categories: javascript
excerpt_separator: <!--more-->
---

Starting out learning javascript can be quite difficult due to some technical terms like single-threaded, non-blocking, asynchronous, callback etc. This is a simple description 9 common javascript terms to help you get started. <!--more-->

### Javascript terms

#### 1. Runtime

Strictly speaking, the runtime refers to the period of time during which the code is running. Runtime environment on the other hand, is the hardware or software infrastructure that is supporting the execution of code and making it run. However, frequently people refer to javascript runtime environment as the javascript runtime. Examples of runtime environments are:
- V8 javascript engine for chrome
- Spideronkey for firefox
- Node for running javascript outside a browser (uses V8)
Simply put, the runtime environment is what handles the execution of javscript code. The browser itself could be said to provide a runtime environment as well, comprised of V8 among other things in the case of chrome.

#### 2. Call Stack

Call stacks are a LIFO (last in first out) data structure used by programming languages to keep track the information about subroutines (or functions). When a program steps into a function the function is pushed on to the stack, and when a function returns it is popped off the stack. This way the program knows where in the code to go back to after completing a function. 

#### 3. Single Threaded

Single threaded is equivalent to having a single call stack. Because there is only one stack the program can handle one thing at a time, or run one piece of code at a time. Only upon completion of a previous piece of code can the next piece of code run.

#### 4. Asynchronous / non-blocking

Being able to only handle one thing at a time is frustrating. The program might get stuck on a bit of code that takes long to execute (for example sending and receiving a request over the internet, or taking user input), other code cannot run until this request is complete. This is called a synchronous code.

In contrast, asynchronous code can be handled by something else, while the main call stack continues with code execution. Asynchronous code does not block the call stack from other pieces of code.

#### 5. Web APIs

Web APIs can handle asynchronous javascript code, and are a part of the runtime environment provided by the browser. If javascript is running on the Node runtime environment, these would be C++ APIs instead. Asynchronous code that uses the web API will be popped off the call stack, so that the call stack can continue with other code. Common examples of web APIs are:
- DOM (Document Object Model) - manipulates the HTML document
- setTimeout - executes code after a set amount of time has passed
- ajax - send and receive data from a server

#### 6. Callbacks

A callback is a function passed to another function as an argument. In terms of asynchronous javascript code, it is the function you want to execute (in the call stack) after the asynchronous code has been completed by the web API. For example if you wanted to increment a counter for each user mouse click, the listening of the mouse click event is handled by the web API, and the increment function would be the callback function to be executed in the call stack when there is a mouse click event.

#### 7. Callback Queue

If the web API directly sends the callback to the main call stack, it could mess up the execution of code happening in the call stack, so it instead sends the callback to the callback queue. When the call stack becomes empty, the callback functions are dequeued onto the call stack to get executed.

#### 8. Render Queue

The render queue is used by the browser when it wants to update how the webpage looks on the browser. A browser often refreshes the page 60 times a second (60Hz), so a request to re-paint the pixels on the browser will be pushed on the render queue 60 times every second.

#### 9. Event Loop

The event loop is what orchestrates the call stack, callback queue and render queue to work together. It checks the render queue if the browser needs to be refreshed. It checks if the call stack is empty, and if it is, it takes tasks from the callback queue to be executed and pushes it on the call stack to execute it. This is why although javascript is single-threaded (only one call stack), it can run asynchronously (let the API handle pieces of code concurrently).

### Further reading

Here are some resources I found extremely helpful when trying to understand this myself:

- [JSConf talk by Philip Roberts](https://www.youtube.com/watch?v=8aGhZQkoFbQ&ab_channel=JSConf)
- [Javascript runtime by Gemma Stiles on Medium](https://medium.com/@gemma.stiles/understanding-the-javascript-runtime-environment-4dd8f52f6fca)
- [Interactive visualization of Javascript runtime](http://latentflip.com/loupe)


[Back](/)
