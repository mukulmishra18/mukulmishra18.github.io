---
layout: post
title: "SendWithStream in PDF.js"
categories: blog
---

## Hi !!

This is second post in the series of my **Google Summer of Code 2017** experience. In the [last post](http://mukulmishra.me/blog/GSoC-First-Post/), I gave a brief introduction of my project. In this post, I am going to talk about [**sendWithStream**](https://github.com/mozilla/pdf.js/blob/master/src/shared/util.js#L1370) method of [**messageHandler**](https://github.com/mozilla/pdf.js/blob/master/src/shared/util.js#L1239).

## What is messageHandler, and how it is used?

PDF.js uses [web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) for parsing PDF commands in background(worker) threads, so that the main thread wouldn't get blocked. At worker thread, PDF commands are parsed and sent to main thread for rendering on HTML canvas. All message passing are done using a common port between the two sides. Here comes `messageHandler` into action, it is a factory, that is used for passing message/data between the two threads. We can listen for an event and trigger some actions using [`_onComObjonMessage`](https://github.com/mozilla/pdf.js/blob/master/src/shared/util.js#L1251) method of _messageHandler_.

For example, say I wanted to setup message passing for `fakeHandler` event, between main and worker threads. I could do it something like:

```javascript
// Setting up messageHandlers on both sides(main + worker).
// Communication is done using a common port(i.e. `fakeport`).
let messageHandler1 = new MessageHandler('main', 'worker', fakePort);
let messageHandler2 = new MessageHandler('worker', 'main', fakePort);

// Listening for `fakeHandler` event at worker thread.
messageHandler2.on('fakeHandler', () => {
  // Do some action here.
  // Send data back to main thread.
});

// Sending `fakeHandler` event from main to worker thread.
messageHandler1.sendWithPromise('fakeHandler', {});
```

## What is sendWithStream, and how it helps?

#### Some basic terminology used in sendWithStream(also defined in [_Streams Spec_](http://streams.spec.whatwg.org/)), we need to understand first:

- **Queuing Strategy**: A _queuing strategy_ is an object(`{ highWaterMark, size }`) that determines how a stream should signal _backpressure_ based on the state of its internal queue.

- **Backpressure**: The process of normalizing flow of chunks from the source to destination is called _Backpressure_. It is used to regulate the speed of flow of chunks, and signal source to slow down the flow when internal queue is full.

- **High Water Mark**: Maximum number of chunks that can be enqueued into the _internal queue_ to fill it completely. When number of chunks in internal queue reached to high water mark, _backpressure_ is signaled to stop further production of chunk from source.

- **Desired Size**: It is the required number of chunks that is needed to fill the internal queue completely. The resulting difference, high water mark minus total size, is used to determine the _desired size_.


#### So how sendWithStream works?

`sendWithStream` method is used to send stream action messages to other side(e.g worker) and store incoming data in stream’s _internal queue_, that can be read using stream’s reader. It returns an instance of `ReadableStream` that can be used to manipulate the flow of data using its **underlyingSources** = `{ start() { }, pull() { }, cancel() { } }`.

- **start**: It is called immediately and used to adapt push source by setting up relevant event listeners. In `messageHandler`, it is used to send message to other side to notify the start of particular action.

- **pull**: It  is called whenever stream’s internal queue of chunks is not full, and will be called repeatedly until the queue reaches its _high water mark_.

- **cancel**: It is used to cancel/close the _underlying source_.

#### [StreamSink](https://github.com/mozilla/pdf.js/blob/master/src/shared/util.js#L1425)??, why it is required?

`StreamSink` is a helper object on the other side(e.g worker) to store sink’s internal state and send message to main side to perform required action. As we don’t have access to stream controller to this side(worker), we can use _streamSink_ to signal controller(at main thread) to perform required action like _enqueue_, _close_, _error_. It also stores sink’s internal state(like _desiredSize_ and _ready_) to block sending enqueue messages when stream’s internal queue is full.

If we look at a minimalistic version of `StreamSink`, it will look something like:

```javascript
let streamSink = {
  enqueue(chunk, size = 1) {
    // Decrese the desiredSize of sink, to regulate enqueue of chunks.
    this.desiredSize -= size;
    // Send message/chunks to main thread, to enqueue into controller.
    sendStreamRequest({ stream: 'enqueue', chunk, });
  },

  close() {
    // Send close message to close controller in main thread.
    sendStreamRequest({ stream: 'close', });
    // Delete streamSink for memory cleanup.
    delete self.streamSinks[streamId];
  },

  error() {
    // Send error message to main thread.
    sendStreamRequest({ stream: 'error', reason, });
  },

  sinkCapability: capability,
  onPull: null,
  onCancel: null,
  desiredSize,
  ready: null,	
};
```
`StreamSink` can be compared to `WritableStream`, as the main motive of creating sink at worker,
is to write into the sink(by calling `sink.enqueue()`) and send the data back to main thread. _StreamSink_ mimic the required functionality of `StreamController` into worker and used to regulate the flow of data in accordance with _controller's_ internal state.

#### Components of StreamSink:**

- **enqueue:** This method is used to enqueue the data into the controller by sending `enqueue` message to main thread. 

- **close:** This method is used to close the controller by sending `close` message to main thread.

- **error:** This method is used to error the controller by sending `error` message to main thread.

- **sinkCapability:** Promise capability{Object} of sink(returned by `createPromiseCapability()`), resolves the capability(by calling`sinkCapability.resolve`) whenever needs.

- **onPull:**(optional) This method calls whenever pull _underlyingSource_ is called.

- **onCancel:**(optional) This method calls whenever cancel _underlyingSource_ is called.

- **desiredSize:** This property defines the number of data chunks required to fill the stream's internal queue completely. Reseted to _desiredSize_ of stream whenever pull is called and decresed whenever enqueue is performed.

- **ready:** This property defines the state of sink(i.e. pending or resolved promise). If `this.desiredSize <= 0` ready is in pending state else ready is resolved. Enqueue is not performed when ready is in pending state.

**Note:** As for now we are unable to send _promises_ and _streams_ to worker thread as tranferable objects, see [this](https://developer.mozilla.org/en-US/docs/Web/API/Worker/postMessage). Maybe in future we can do so, but for now we have to support old browsers.

Follow [sendWithStream PR](https://github.com/mozilla/pdf.js/pull/8430) for all the discussions or read the full code [here.](https://github.com/mukulmishra18/pdf.js/commit/bbd9968f76c68f6120a6e36825796347b7bb152a)