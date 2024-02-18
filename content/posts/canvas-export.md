+++
title = 'Exporting HTML canvas to PDF & PNG using JS'
date = 2024-02-17T16:33:45+08:00
categories = ['Canvas']
+++

When exporting the content of a whiteboard web app, we are exporting the content of an HTML canvas element to PDF format(Vector Image) or PNG(Raster Image). This post will introduce the basic method for exporting the content of HTML canvas and also some issues we encountered when working on this feature.

<!-- more -->

## Export to PDF

### Export method

We'll use [PDFKit](https://pdfkit.org/) to generate the PDF file. We have the SVG string as the source for the vector format of the content we draw on the HTML canvas element. 

So we need a tool to convert SVG to PDFKit command. That is [svg-to-pdfkit](https://github.com/alafr/SVG-to-PDFKit). Unfortunately, this lib seems no longer maintained, so it has some drawbacks. And we may need to fork this lib to meet certain requirements.

### CJK font issue

When implementing this feature, we found PDFKit unable to render non-ASCII characters. This happens because PDFKit needs a font file to render all the characters to paths and it only has a small built-in font file that cannot cover all the characters. And we need to add some external font files to support all the characters.

To inject the font file into PDFKit, we download the font file from the network and call `registerFont`.

```js
const fontUrl = "https://fonts.com/noto-sans-sc.woff2";

fetch(fontUrl).then(res => res.arrayBuffer()).then(buffer => {
  doc.registerFont('Noto Sans SC', buffer)
  doc.font('Noto Sans SC')
     .text('你好，世界!')
});
```

In most cases, this will work. But for the CJK languages, we have separate font files for different languages with the same font family. Such as Noto Sans SC/Noto Sans JP/Noto Sans KR.

What we need to consider next is how to determine the font family for each string. One solution is to combine all the font files into one file. But this may slow down the download speed when the user does not have any CJK characters in their content. 

The other solution is to loop all the strings and check each character's glyph script set. And add the font family attribute to the SVG tag on the fly. This method also has some drawbacks. We use regex to match script type, but we also need to match some punctuation only used in CJK languages, such as [fullwidth parenthesis](https://www.compart.com/en/unicode/U+FF09). This character is only included in the CJK fonts. But the script property for this character belongs to "Common".  So it is difficult to use a simple regex to determine the correct font type for a character.

So combining font files may be a better solution if we want the PDF render result to be perfect.

## Export to image

### Export method

Image exporting is more straightforward. We just call create a canvas element, render the content and call `canvas.toDataURL("image/png")`. Finally, we add the data URL to a link to download the image.

### Canvas max size issue

The issue arises when we have a very big canvas to export. Safari has a [small limit for canvas size](https://pqina.nl/blog/canvas-area-exceeds-the-maximum-limit/). 

What's worse, Safari will fail silently when the canvas exceeds the maximum limit. This limit is different for each type of device due to the different total memory they have. So we need to detect the maximum limit and resize the canvas in advance. We can use [this lib](https://jhildenbiddle.github.io/canvas-size/#/) to detect the maximum limit on the fly. In my opinion, we only need to check this on Safari. Chromium-based browsers always have the same limit.

<!-- ## Export to CSV file

### BOM header issue -->