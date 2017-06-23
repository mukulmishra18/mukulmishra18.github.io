---
layout: post
title: "Streaming PDF TextContent"
categories: blog
---

## Hello,

This is third post in the series of my **Google Summer of Code 2017** experience. In [last post](http://mukulmishra.me/blog/sendWithStream-in-PDF.js/), i gave detailed overview of _sendWithStream_ method of _messageHandler_. In this post, i am going to give updates of [my project](https://github.com/mozilla/pdf.js/projects/4) _Streams API in PDF.js_.

First phase of coding is about to end, and we already have our first Streams API supported PDF.js API i.e. **streamTextContent**. This API is inspired from PDF.js [**getTextContent**](https://github.com/mozilla/pdf.js/blob/master/src/display/api.js#L958) API, that is used to extract text contents of PDFs. As name suggests, _streamTextContent_ can be used to stream text contents of PDFs incrementally in small chunks.

#### So how it is going to help?

Earlier we are using _getTextContent_ API to retrieve text content of PDFs, that is not very efficient in terms of memory and speed. When _getTextContent_ requests text content from worker thread, it has to wait unit all the text contents are build. Text content is accessible in main thread only when it is completely build at worker thread and send via _MessageHandler_. Due to this reason, it is creating inefficiency in terms of:

- **Speed**: As we have to wait for building the whole text content.

- **Memory**: As we have to store all the text contents in worker thread and send in bulk.


But using _streamTextContent_ will solve this problem, as we can stream text contents in small chunks whenever we have any in worker thread. This will eliminate the inefficiency in terms of:

- **Speed**: As now, we don't have to wait in main thread for text contents and be able to start rendering process incrementally.

- **Memory**: As now, we don't have to store whole text content in worker thread, and be able to stream text contents in small chunks.

#### So how _streamTextContent_ works?

I would recommend to read my [last post](http://mukulmishra.me/blog/sendWithStream-in-PDF.js/) first, it will explain how we are streaming data between the two threads(main + worker).

