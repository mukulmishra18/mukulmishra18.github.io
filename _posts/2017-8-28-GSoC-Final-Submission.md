---
layout: post
title: "GSoC 2017: Final Submission"
categories: blog
---

## Hello !!

In this post, I am going to give a report on my project [Streams API in PDF.js](https://summerofcode.withgoogle.com/projects/#5056427950342144), done as a part of _Google Summer of Code 2017_ program. I sincerely thank everyone in PDF.js community, including my mentor [Yury Delendik](https://github.com/yurydelendik) and members of PDF.js project [Jonas Jenwald](https://github.com/Snuffleupagus), [Tim Van der Meij](https://github.com/timvandermeij) and [Rob Wu](https://github.com/Rob--W), for all their guidance during my project.

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
- [GSoC first post, getting started with PDF.js](http://mukulmishra.me/blog/GSoC-First-Post/)
- [sendWithStream in PDF.js](http://mukulmishra.me/blog/sendWithStream-in-PDF.js/)
- [Streaming PDF Text Content](http://mukulmishra.me/blog/Streaming-PDF-TextContent/)
- [PDF.js Network Streaming](http://mukulmishra.me/blog/PDF.js-Network-Streaming/)
- [PDF.js Node Stream](http://mukulmishra.me/blog/PDF.js-Node-Stream/)
- [PDF.js Fetch Stream](http://mukulmishra.me/blog/PDF.js-Fetch-Stream/)
