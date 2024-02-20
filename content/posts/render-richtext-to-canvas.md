+++
title = 'A quick way to layout & render HTML rich text to canvas'
date = 2024-02-17T16:33:45+08:00
+++

As we know, the canvas `fillText` API can only draw a single line of text. If we want to draw multiple lines of text, or we want to draw a line of text with different kinds of styles and font sizes, we'll need to find a way to layout all the text blocks. That is, try to figure out where to put the text and get the x and y parameters for the `fillText` API.

<!-- more -->

We can render the rich text in HTML using DOM. And we can use `html2canvas` lib to render some DOM nodes into canvas. Problem solved!🤣 

It's fine to use `html2canvas`. But in this post, we'll introduce a simple and fast way to draw DOM text nodes into the canvas without using any libs.

I find the clue from [this blog using Range API](https://www.bennadel.com/blog/4310-detecting-rendered-line-breaks-in-a-text-node-in-javascript.htm) to get the BoundingRect for each line of text, `html2canvas` also uses this API. But this blog uses a brute force algorithm to get all the BoundingRects. We can always do better than brute force. 

So I came up with a step-based way to find all the line boxes.

<iframe width="100%" height="500px" src="https://stackblitz.com/edit/vitejs-vite-rzpfqh?embed=1&file=src%2Fmain.ts"></iframe>

At first, we can get all line boxes of a text node using one `getClientRects` call by setting a range that starts from 0 and ends at the last index of the string.

When we traverse all the line boxes, we can get each line box's width. We can estimate the character count of each line by calculating:

`(the line box's width / sum of all line box's width) * textNode's text length`

And we count this value for each line box and we use this array as some sort of step.

When we try to find the line breaks, instead of adding one index to the range's end each time, we'll add the value in the step array for the corresponding line index to the range's end. After that, we'll search by one step to get the precise position of the line break.

> Note that for RTL language, we have to use the line box order.