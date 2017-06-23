---
layout: post
title: "Streaming PDF TextContent"
categories: blog
---

## Hello,

This is third post of my **Google Summer of Code 2017** experience. In [last post](http://mukulmishra.me/blog/sendWithStream-in-PDF.js/), i gave detailed overview of _sendWithStream_ method of _messageHandler_. In this post, i am going to give updates of [my project](https://github.com/mozilla/pdf.js/projects/4) _Streams API in PDF.js_.

First phase of coding is about to end, and we already have our first streaming PDF.js API i.e. **streamTextContent**. This API is inspired from PDF.js [**getTextContent**](https://github.com/mozilla/pdf.js/blob/master/src/display/api.js#L958) API, that is used to extract text contents of PDFs. As name suggests, _streamTextContent_ can be used for stream text contents of PDFs incrementally in small chunks.
