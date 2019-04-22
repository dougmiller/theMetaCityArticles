Title: Lets make a terrible image processing pipeline
Blurb: With the advent of the `<picture>` tag and the general support of media-queries, it is time to automate image production.
Tags: Automation,Images

It used to be since [way back in the day](http://1997.webhistory.org/www.lists/www-talk.1993q1/0182.html) that if you want to put an image on a page then you put the image on the page and users be dammed if it didnt work for their screen.

While people had previously broached the topic of media-queries it took until the early 2000's to get an agreed up spec drafted and then more than another decade to have it formalised after enough browser by in happened.

With the arival of mobile and HTML5 the door opened again as people began to look at ways to combat Wirth's Law slowing down user experience's and download times.

Two things emerged out of this: the `<picture>` tag with media queries for different resolutions and newer, more effecient image formats. Combined, users only need to download the images at the resolution they will actually use and in the most effecient format they will use. Win, win.

It is not all peaches and cream however as this:

```html
<img src="https://example.com/logo.png" alt="Example site's logo, rainbow coloured and flying high" title="See example site's page">
```
becomes:

```html
<picture>
    <source srcset="https://example.com/logo_274-37.webp 274w,
    https://https://example.com/logo_160-22.webp 160w"
    sizes="(max-width: 767px) 160px,
            274px"
    type=image/webp>
    
    <source srcset="https://example.com/logo_274-37.png 274w,
    https://https://example.com/logo_160-22.png 160w"
    sizes="(max-width: 767px) 160px,
            274px"
    type=image/png>
    <img src="https://example.com/logo.png" alt="Example site's logo, rainbow coloured and flying high" title="See example site's page">
</picture>
```

So a bit more work needs to go into producing a page, the logistics of which are for a different article [i.e](/blog/lets-make-a-terrible-markdown-extension-pt1-background). I am going to go through how I automated the creation of the images that feed into the `srcset`s.

## First things first

To make this as lazy and as efficient as possible, the solution should probably revolve around a pipeline that watches a folder, does the work and then spits it out in a known folder. No clicking, no selecting and image then pressing upload and then download a zip of the contents. No waiting till I am back online after stomping around offline for a few days.

Local folders, local solutions.

[This screams of `inotifywait`](https://linux.die.net/man/1/inotifywait).

## Second things second
There are a few things that will need to happen in this pipeline: take in an image, convert it to the defined formats, resize it to fit the media queries, rename the files appropriatly and then put them somewhere.

This whole process could be one giant bash script but that sounds a nightware to write, debug and support. So taking a page out of the unix philosophy: one function, one folder.

The general approach is: have a folder for the function (`format`, `resize`, `rename` etc) which can watch for input (a file written or copied in), do the next step and then push the file out to the next function. KISS.

## `inotifywait`
You can ask linux to watch and react to a pretty compehansive set of actions that might happen on a folder or file: running `inotifywait -m file` will output them as they come in (the '`-m`' is for `monitor` which is used as the program will stop after the first event otherwise.

This will output  






