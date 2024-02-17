+++
title = 'Exporting HTML canvas to PDF & PNG using JS'
date = 2024-02-17T16:33:45+08:00
+++

When exporting the content of a whiteboard web app, we are exporting the content of a HTML canvas element to PDF format(Vector Image) or PNG(Raster Image). This post will introduce the basic method for exporting content of HTML canvas and also some issues we encountered when working on this feature.

<!-- more -->

## Export to PDF

### Export method

We'll use PDFKit to generate the PDF file. And we have the svg string as the source for the vector format of the content we draw on HTML canvas element. 

So we need a tool to convert SVG to PDFKit command. That is svg-to-pdfkit. Unfortunely, this lib is no longer maintained, so it has some drawback. And we may need to fork this lib to meet certain requirements.

### CJK font issue

When implement this feature.

## Export to image

### Export method

### Canvas max size issue

## Export to CSV file

### BOM header issue