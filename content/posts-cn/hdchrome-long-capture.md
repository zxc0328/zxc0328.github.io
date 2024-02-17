---
title: "Headless Chrome long image capture issue"
date: 2018-02-12 11:16:21
tags: 
- Headless Chrome
categories: 
- Headless Chrome
---


### The problem

Recently I had received complaint about my capture service not export complete image. It seems that this problem only occurs when the page's is extremely long.

<!-- more -->

The broken image is like this:

![broken](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fmlvek2n5xj208y07aaah.jpg)


### Chromium's limit

So I Googled for the problem and I found [a lot issues](https://github.com/GoogleChrome/puppeteer/issues/477) on Github that target the same problem. When reading throught [this issue](https://github.com/GoogleChrome/puppeteer/pull/937), I got the fact that this problem is caused by Chromium's limit.

Since normal server don't have a GPU inside, Headless Chrome had to use software renderer, that is, using CPU to calculate the pixels. 

Chromium's compositor has a maximum texture size when using software GL backend, this limit is 16384px. So large image will not be renderer completely.

### How to solve it

The solve for this problem is simple. Cut the page into pieces, capture these fragments in order, and composite those pieces into a whole image.

The code below use Puppeteer's API, it's fine to replace it with other library like CDP.

```
await page.setViewport({ width: 1440, height: 1024});
const {contentSize} = await page._client.send('Page.getLayoutMetrics');
// MAGIC NUMBER, DO NOT MODIFIY THIS OR YOU WILL BE FIRED
const maxScreenshotHeight = 7000;
          if (contentSize.height >= maxScreenshotHeight) {
                        
            let image;
            let lastBuffer;

            for (let ypos = 0; ypos < contentSize.height; ypos += maxScreenshotHeight) {
              const height = Math.min(contentSize.height - ypos, maxScreenshotHeight);
              let buffer = await page.screenshot({
                clip: {
                  x: 0,
                  y: ypos,
                  width: contentSize.width,
                  height
                }
              });
              if (ypos === 0) {
                image = sharp(buffer);
                lastBuffer = await image.toBuffer();
              }else {
                image = sharp(lastBuffer);
                image = image.extend({top: 0, bottom: height, left: 0, right: 0})
                image = image.overlayWith(buffer, {top: ypos, left:0})
                lastBuffer = await image.toBuffer();
              }
            }
            fileData = lastBuffer;
```

I use [sharp](https://github.com/lovell/sharp) for image processing, bacause it's recommended on Github issue.


> The magic number here should be around 16000 by default, but I had noticed that the height for each piece should be reduced when the width increased. The width I use for my capture is 1440px, you should start from 16000 and reduce this number if the image exported is still incomplete.

### Future

The approach may not be necessary accroding to this [Chromium issue](https://bugs.chromium.org/p/chromium/issues/detail?id=770769).