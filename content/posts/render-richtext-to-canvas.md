+++
title = 'A quick way to layout & render HTML rich text to canvas'
date = 2024-02-17T16:33:45+08:00
draft = true
+++

As we know, the canvas `fillText` API can only draw a single line of text. If we want to draw multiple lines of text, or we want to draw a line of text with different kinds of styles and font sizes, we'll need to find a way to layout all the text blocks. That is, to figure out where to put the text, the x and y parameters for the `fillText` API.

<!-- more -->

We can render the rich text in HTML using DOM. And we can use `html2canvas` lib to render some DOM nodes into canvas. Problem solved!🤣 

It is totally fine to use `html2canvas`. But in this post, we'll introduce a simple and fast way to draw DOM text nodes into canvas without using any libs.

I find the clue from [this blog using Range API](https://www.bennadel.com/blog/4310-detecting-rendered-line-breaks-in-a-text-node-in-javascript.htm) to get the BoundingRect for each line of text, `html2canvas` also use this API. But this blog use a brute force algorthm to get all the BoundingRects. We can always do better than brute force.

So I came up a step-based way to find all the line boxes.

// todo: add the code sandbox here.

> RTL issue.