---
layout: post
title: "Streaming PDF TextContent"
categories: blog
---

## Hello !!

This is the third post in the series of my **Google Summer of Code 2017** experience. In [last post](http://mukulmishra.me/blog/sendWithStream-in-PDF.js/), I gave detailed overview of `SendWithStream` method of `MessageHandler`. In this post, I am going to give updates of [my project](https://github.com/mozilla/pdf.js/projects/4) _Streams API in PDF.js_.

First phase of coding is about to end, and we already have our first Streams API supported PDF.js API, i.e., **streamTextContent**. This API is inspired from PDF.js [**getTextContent**](https://github.com/mozilla/pdf.js/blob/master/src/display/api.js#L958) API, that is used to extract text contents of PDFs. As name suggests, _streamTextContent_ can be used to stream text contents of PDFs incrementally in small chunks.

#### So how it is going to help?

Earlier we were using `getTextContent` API to retrieve text content of PDFs, that is not very efficient in terms of memory and speed. When _getTextContent_ requests text content from worker thread, it has to wait until all the text contents are built. Text content is accessible in main thread only when it is completely built at worker thread and send via `MessageHandler`. Due to this reason, it creates inefficiency in terms of:

- **Speed**: As we have to wait for building the whole text content.
- **Memory**: As we have to store all the text content in worker thread and send in bulk.


But using `streamTextContent` will solve this problem, as we can stream text contents in small _chunks_ whenever we have any in worker thread. This will eliminate the inefficiency in terms of:

- **Speed**: As now, we don't have to wait in main thread for text contents and we would be able to perform rendering process incrementally.
- **Memory**: As now, we don't have to store whole text content in worker thread, and be able to stream text contents in small chunks.

#### So how _streamTextContent_ works?

I would recommend to read my [last post](http://mukulmishra.me/blog/sendWithStream-in-PDF.js/) first, it will explain how we are streaming data between the two threads(main + worker).

**Let's first understand how _getTextContent_ works:**

When `getTextContent` API is called at main thread, it sends message to worker using `MessageHandler`.
This message is handled by _[GetTextContent](https://github.com/mozilla/pdf.js/blob/master/src/core/worker.js#L877)_ handler in _worker.js_ file. Parsing of PDF commands are done in _evaluator.js_, `GetTextContent` handler calls `getTextContent` method of `PartialEvaluator` via [_document.js_](https://github.com/mukulmishra18/pdf.js/commit/0c13d0ff4618b3759d84a506390e31b573d7221d#diff-12f6a5494fae2f6d944246a1830f8982R302) file. In _getTextContent_ method of _PartialEvaluator_, whole [textContent](https://github.com/mozilla/pdf.js/blob/master/src/core/evaluator.js#L1188) is build and send back to main thread by resolving promises.

Building whole text content at worker and sending back to main thread in bulk take lots of memory. Text content is only available at main thread, when promise at `PartialEvaluator` is resolved(i.e. `resolve(textContent)`), that forces rendering processes to wait. This degrade user experience(by janky scrolling, slow rendering...).


**Incrementally sending chunks using streamTextContent**

The above mentioned problems can be eliminated by incrementally sending data chunks from worker to main thread, and rendering PDF on the fly. In _streamTextContent_ method, instead of building the whole text content in woker, we are sending it to main thread whenever we have any.

In the [`next`](https://github.com/mozilla/pdf.js/blob/master/src/core/evaluator.js#L1433) function of `PartialEvaluator`, we are calling `enqueueChunk()` to send back data chunks to main thread. If we look into the _enqueueChunk_ function, it is self explanatory:

```javascript
function enqueueChunk() {
  let length = textContent.items.length;
  if (length > 0) {
    // Enqueue chunks to sink.
    sink.enqueue(textContent, length);
    // Reset textContent to free memory.
    textContent.items = [];
    textContent.styles = Object.create(null);
  }
}
```

_next_ function is best suited for calling _enqueueChunk_, because it is called whenever any promise needs to be resolved. Basically it holds the parsing process for sometimes, and that is best time to enqueue chunks in sink, e.g:

```javascript
next(promise) {
  enqueueChunk();
  promise.then(() => {
    // Resume parsing process.
  });
}
```

**Using _streamTextContent_ internally in _getTextContent_**

For not breaking the support of `getTextContent` API, we have created a parallel method `streamTextContent` to support Streams API in the process of retrieving the text content of PDFs. But it is better idea to use _streamTextContent_ internally in _getTextContent_, so that we can stream text content in that too. Using `readAllChunk` approach in _getTextContent_, will refactor the API something like:

```javascript
getTextContent() {
  params = params || {};
  let readableStream = this.streamTextContent(params);

  return new Promise(function(resolve, reject) {
    function pump(params) {
      reader.read().then(function({ value, done, }) {
        if (done) {
          resolve(textContent);
          return;
        }
        // Build text content by looping through all read operations.
        Util.extendObj(textContent.styles, value.styles);
        Util.appendToArray(textContent.items, value.items);
        pump();
      }, reject);
    }

    let reader = readableStream.getReader();
    let textContent = {
      items: [],
      styles: Object.create(null),
    };

    pump();
  });
}
```

#### Some changes to PDF.js viewer to use _streamTextContent_

Some changes has made into the PDF.js standard [viewer](https://mozilla.github.io/pdf.js/web/viewer.html) to use _streamTextContent_ instead of _getTextContent_, so that we can measure live performance of Streams API support.

For the same, we have passed the _RedableStream_ from [pdf_page_view.js](https://github.com/mukulmishra18/pdf.js/commit/0c13d0ff4618b3759d84a506390e31b573d7221d#diff-c880b70cf7d6b97863eeb665a736c4baR448) to text_layer.js via [text_layer_builder.js](https://github.com/mukulmishra18/pdf.js/commit/0c13d0ff4618b3759d84a506390e31b573d7221d#diff-6a8d3c6754c77cfa0876314f0f54bed7R93). Reading chunks in [`_render`](https://github.com/mukulmishra18/pdf.js/commit/0c13d0ff4618b3759d84a506390e31b573d7221d#diff-986df9c532f6b9ab6cf81a23ce26e446R556) method of `TextLayerRenderTask`, and passing incrementally to [`_processItems`](https://github.com/mukulmishra18/pdf.js/commit/0c13d0ff4618b3759d84a506390e31b573d7221d#diff-986df9c532f6b9ab6cf81a23ce26e446R512) will lower memory usage of viewer. Cancelling the _read_ operation of stream in [`cancel`](https://github.com/mukulmishra18/pdf.js/commit/0c13d0ff4618b3759d84a506390e31b573d7221d#diff-986df9c532f6b9ab6cf81a23ce26e446R499) method when page is not into view, make viewer more responsive. 

#### Performance measurement of streamTextContent:

**Steps for measuring the performance:**

- Open the viewer in master branch, and paste [this](https://gist.github.com/mukulmishra18/831e74fc0d49398f3172c5af0c6c4afc) code in console to take data for automation of scrolling.
- Paste above taken data(or [this](https://gist.github.com/mukulmishra18/784136c6b75dacf6043c38d18039d0d5)) into the console(as `rr = data`), and run [this](https://gist.github.com/mukulmishra18/19e53e31fb89ce704866a7e2ad06d3a5) code in console to perform automatic viewer scrolling.
- Measure the memory usage using devtools or run [this](https://gist.github.com/mukulmishra18/18635de4c82522bde3e7c4c0761f3365) code as `python name_of_file.py <pID>`, where pID is process ID of viewer. This will give peak memory used, when process ends.


**Measurement Configuration:**

- Operating system: Ubuntu 14.04 LTS
- Browser: Chrome (version: 58.0.3)
- PDF: [http://www.pharmazeutische-zeitung.de/fileadmin/pdf/PZ-Beilage-ApBetrO_2012_21.pdf](http://www.pharmazeutische-zeitung.de/fileadmin/pdf/PZ-Beilage-ApBetrO_2012_21.pdf)
- script[1]: [https://gist.github.com/mukulmishra18/9954e1e2c3c254c77b69c8985fe97049](https://gist.github.com/mukulmishra18/9954e1e2c3c254c77b69c8985fe97049)

**Output:**

| Branch\measurement[2] | 1 	 | 2	  | 3	   | 4	    | 5 	 | 6	  |
| :-------------------- | :----: | :----: | :----: | :----: | :----: | -----: |
| master 			    | 399.39 | 397.07 | 416.43 | 405.59 | 433.07 | 428.06 |
| streams 				| 390.02 | 389.16 | 385.09 | 384.05 | 387.34 | 387.46 |
{: .table}


| Branch\measurement[3] | 1     | 2     | 3     |
| :-------------------- | :---: | :---: | ----: |
| master 				| 16.74 | 16.56 | 16.81 |
| streams 				| 17.44 | 18.07 | 18.77 |
{: .table}

<br />
[1] Run script in web console to perform automatic scrolling.<br />
[2] Memory used(in MB) during measurement.<br />
[3] Average fps during measurement.<br />
<br />

**Result:**

From the above measurements, we can conclude that we are using comparatively less memory using Streams API in PDF.js viewer. Using less memory and cancelling streaming of text content chunks, when page scrolled out of view, makes viewer more responsive.
<br />

**Measurement snapshorts**
<figure>
	<a href="http://i.imgur.com/dPsVzMc.png"><img src="http://i.imgur.com/dPsVzMc.png" alt="measurement-with-stream"></a>
	<figcaption><a href="http://i.imgur.com/dPsVzMc.png" title="FPS(frame per second) graph for streaming data"></a>FPS(frame per second) graph for streaming data</figcaption>
</figure>

<figure>
	<a href="http://i.imgur.com/AOx24en.png"><img src="http://i.imgur.com/AOx24en.png" alt="measurement-without-stream"></a>
	<figcaption><a href="http://i.imgur.com/AOx24en.png" title="FPS(frame per second) graph for non streaming data"></a>FPS(frame per second) graph for non streaming data</figcaption>
</figure>


<br />
Follow [streams-getTextContent PR](https://github.com/mozilla/pdf.js/pull/8488) for all the discussions or read the full code [here.](https://github.com/mukulmishra18/pdf.js/commit/0c13d0ff4618b3759d84a506390e31b573d7221d)