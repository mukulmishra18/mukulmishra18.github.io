---
layout: post
title: "GSoC 2017: Final Submission"
categories: blog
---

## Hello !!

In this post, I am going to give a report on my project [Streams API in PDF.js](https://summerofcode.withgoogle.com/projects/#5056427950342144), done as a part of _Google Summer of Code 2017_ program. I sincerely thank everyone in PDF.js community, including my mentor [Yury Delendik](https://github.com/yurydelendik) and members of PDF.js project [Jonas Jenwald](https://github.com/Snuffleupagus), [Tim Van der Meij](https://github.com/timvandermeij) and [Rob Wu](https://github.com/Rob--W), for all their guidance during my project.

#### Abstract:
This project aims to implement Streams API polyfill for PDF.js, and use it for networking and rendering purpose. PDF.js current uses XHR for networking to fetch data, but it is inefficient in terms of memory as it has to buffer all the response. As XHR is an ever growing buffer, it stores all the data in its internal buffer, this uses lots of memory. Using [Streams API](https://streams.spec.whatwg.org/), we can use the response incrementally and pass it for parsing and rendering as soon as it is received, hence there is no need to buffer the whole response. For rendering purpose, PDF.js makes chunks of data by parsing pdf commands in worker thread. After chunking the whole data, it is passed to the main thread for painting. Buffering the entire data in the worker thread uses a huge amount memory, and main thread also has to wait for the entire data to be parsed. This creates inefficiency in terms of memory and speed. Using Streams API, we can send the chunks from worker thread to main thread as soon as they are created, reducing the memory usage and increases speed.     


#### Milestones for the project:
- Streams API polyfill to support legacy browsers
- Implement sendWithStream method in MessageHandler to stream data between main and worker thread
- Support Streams API in getTextContent API of PDF.js to implement streamTextContent
- Some changes to use streamTextContent in PDF.js Viewer
- Support Streams API in getOperatorList API of PDF.js to implement streamOperatorList
- Some changes to use streamOperatorList in PDF.js Viewer
- Move networking part of PDF.js from worker to main thread
- Implement node_stream.js to support streams in node environment
- Implement fetch_stream.js to support streams in browsers

#### Achieved Milestones:
- [Streams API polyfill to support legacy browsers](https://github.com/mozilla/pdf.js/pull/8396)
- [Implement sendWithStream method in MessageHandler to stream data between main and worker thread](https://github.com/mozilla/pdf.js/pull/8430)
- [Support Streams API in getTextContent API of PDF.js to implement streamTextContent](https://github.com/mozilla/pdf.js/pull/8488)
- [Some changes to use streamTextContent in PDF.js Viewer](https://github.com/mozilla/pdf.js/pull/8488)
- [Move networking part of PDF.js from worker to main thread](https://github.com/mozilla/pdf.js/pull/8617)
- [Implement node_stream.js to support streams in node environment](https://github.com/mozilla/pdf.js/pull/8712)
- [Implement fetch_stream.js to support streams in browsers](https://github.com/mozilla/pdf.js/pull/8768)

#### Work left to be done:
- Support Streams API in getOperatorList API of PDF.js to implement streamOperatorList
- Some changes to use streamOperatorList in PDF.js Viewer

#### Link to blog post related to the project:
- [GSoC first post, getting started with PDF.js](https://mukulmishra18.github.io/blog/GSoC-First-Post/)
- [sendWithStream in PDF.js](https://mukulmishra18.github.io/blog/sendWithStream-in-PDF.js/)
- [Streaming PDF Text Content](https://mukulmishra18.github.io/blog/Streaming-PDF-TextContent/)
- [PDF.js Network Streaming](https://mukulmishra18.github.io/blog/PDF.js-Network-Streaming/)
- [PDF.js Node Stream](https://mukulmishra18.github.io/blog/PDF.js-Node-Stream/)
- [PDF.js Fetch Stream](https://mukulmishra18.github.io/blog/PDF.js-Fetch-Stream/)
