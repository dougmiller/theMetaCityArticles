Title: Lets make a terrible image processing pipeline
Blurb: With the advent of the `<picture>` tag and the general support of media-queries, it is time to automate image production.
Tags: Automation,Images

It used to be since [way back in the day](http://1997.webhistory.org/www.lists/www-talk.1993q1/0182.html) that if you want to put an image on a page then you put the image on the page and users be dammed if it didn't work for their screen.
While people had previously broached the topic of media-queries it took until the early 2000's to get an agreed up spec drafted and then more than another decade to have it formalised after enough browser buy in happened.

With the arrival of mobile and HTML5 the door opened again as people began to look at ways to combat Wirth's Law slowing down user experience's and download times.

Two things that emerged out of this: the `<picture>` tag with media queries for different resolutions and newer, more efficient image formats. Combined, users only need to download the images at the resolution they will actually use and in the most efficient format they will use. Win, win.

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

Local problems, local solutions.

[This screams of `inotifywait`](https://linux.die.net/man/1/inotifywait).

## Second things second
There are a few things that will need to happen in this pipeline: take in an image, convert it to the defined formats, resize it to fit the media queries, rename the files appropriately and then put them somewhere.

This whole process could be one giant bash script but that sounds a nightmare to write, debug and support. So taking a page out of the Unix philosophy: one function, one folder.

The general approach is: have a folder for the function (`format`, `resize`, `rename` etc) which can watch for input (a file written or copied in), do the next step and then push the file out to the next function. KISS.

## `inotifywait`
You can ask Linux to watch and react to a pretty comprehensive set of actions that might happen on a folder or file: running `inotifywait -m file` will output them as they come in (the '`-m`' is for `monitor` which is used as the program will stop after the first event otherwise.

This will output all the events that happen to the file or folder that is monitored.

```bash
~> inotifywait -m Desktop/drop/
Setting up watches.
Watches established.
Desktop/drop/ CREATE foo.png
Desktop/drop/ OPEN foo.png
Desktop/drop/ MODIFY foo.png
Desktop/drop/ MODIFY foo.png
Desktop/drop/ MODIFY foo.png
Desktop/drop/ MODIFY foo.png
Desktop/drop/ MODIFY foo.png
Desktop/drop/ MODIFY foo.png
Desktop/drop/ CLOSE_WRITE,CLOSE foo.png
Desktop/drop/ OPEN,ISDIR 
Desktop/drop/ ACCESS,ISDIR 
Desktop/drop/ ACCESS,ISDIR 
Desktop/drop/ CLOSE_NOWRITE,CLOSE,ISDIR 
Desktop/drop/ OPEN,ISDIR 
Desktop/drop/ ACCESS,ISDIR 
Desktop/drop/ CLOSE_NOWRITE,CLOSE,ISDIR 
Desktop/drop/ MOVED_FROM foo.png
```

You can see that I copied the file `foo.png` into the folder (moved has a different event `MOVED_TO`), the system opens the file, copied the content in and then closed it. The system then accessed it a bit later and finally I moved the file out.

The output can be formatted thusly: `inotifywait --format <format>`

```bash
inotifywait -m Desktop/drop/ --format %w%f
Setting up watches.
Watches established.
Desktop/drop/foo.png
```

You can listen to only the events you want too: `inotifywait -e <event[,event,...]>`

```bash
~>inotifywait -m Desktop/drop/ -e moved_to
Setting up watches.
Watches established.
Desktop/drop/ MOVED_TO foo.png
```

Very handy stuff. The `man` page has the details you need.

## Of note
While up the actual watches and processes is fairly straightforward at this point there are a few gotchas.

The `create` event does not mean the file is ready to use. For example, when the file conversion for the `webp` converter runs, it will make the new file directly where it is told (rather than a temp file in a temp folder and move it in when done). This will trigger a `create` event then start filling it with the content of the file before firing off a `close` event (`close_write` specifically). Moving a file only triggers the move event. Be aware of what you are doing and watching for.

Running multiple `inotifywait` from the one script gets tricky. Running it will block and wait for it to return before allowing the rest of the script to run. You will need to run it in the background (`&`).


## Get on with it already.
### Entry
To get files into the process, the easiest way would be to watch a known folder, take the files dumped and then put them into the next step. Not much to this one.

```bash
inotifywait -m <watched_folder> -q --format '%w%f' -e close_write,moved_to | \
    while read file; do
        cp ${file} finished # Original file
        mv ${file} formatter/drop
    done
```

As mentioned previously, we are watching for files both copied in and also moved in. Formatter is the next step in the process. The `cp` line takes the original file and copies it to the end of the process as an unmolested original, the dropped file then enters into the formatting process.

### Formatter

```bash
formats=$(find . -type d -not -path . -not -path ./drop -not -path ./finished)

inotifywait -m drop -q --format '%w%f' -e moved_to | \
    while read file; do
        while read -r dir; do
            cp ${file} ${dir}
        done <<< ${formats}
        cp ${file} ../resizer/drop # need to convert original format too
        rm ${file}
```

There are several folders here that need to be looked at:
```bash
converter_flif.sh  drop     finished     flif    webp
converter_webp.sh  drop.sh  finished.sh  run.sh
```
The drop and finished (and their `.sh` contemporaries) are just the entry and exit points to the step. The others are the ones that do the work. Each is a format that gets converted to. It is assumed that `jpg`s do not get converted to `png`s and vice-a-versa so no folder for them). While `webp` will consume most anything you can throw at it, `flif` is not nearly so mature and will only do `png`'s (and other vector image types) at time of writing. `jpg`s will still end up in the folder but the conversion will fail and there will be no output.

```bash
# Relying on silently swallowing errors as to weather or not the conversion was a success
# Currently only PNG are supported. JPGs will just be swallowed.

if [[ ! -d flif ]]; then
    mkdir flif
fi

inotifywait -m flif -q --format '%f' -e close_write | \
    while read file; do
        flif flif/${file} ../finished/${file%.*}.flif &> /dev/null
        rm flif/${file}
    done
```

Due to the ability for flif to only download the data needed to show well at the required size, this output goes directly to finished. The other(s) go to the `resize` step.

<aside>You will see this pattern of watching a drop folder and coping it to a finished folder which moves it to the next folder a lot. It seems to work quite well.</aside>

### Resizer

```bash
#!/usr/bin/env bash

source ../functions.sh

size_folders=$(find . -type d -not -path . -not -path ./drop -not -path ./finished)

inotifywait -m -q -r ${size_folders} --format '%w%f' -e close_write | \
    while read found_file; do
        file_name=$(echo ${found_file} | rev | cut -d '/' -f 1 | rev | cut -d '.' -f 1)
        file_ext=$(echo ${found_file} | rev | cut -d '/' -f 1 | rev | cut -d '.' -f 2)
        file_type=$(get_type ${found_file})
        size_shape=$(echo ${found_file} | rev | cut -d '/' -f 2 | rev)
        shape=${size_shape: -1}
        size=${size_shape::-1}

        # 'convert' needs XSIZExYSIZE and 'cwebp' needs XSIZE YSIZE so doubling up variables to use this later
        if [[ ${shape} = 'x' ]]; then
            direction=''
            x=${size}
            y=0
        else
            direction=x
            y=${size}
            x=0
        fi

        if [[ ${file_type} = 'jpg' ]]; then
            convert ${found_file} -resize ${direction}${size} finished/${file_name}_${size_shape}.${file_ext} &> /dev/null
        fi

        if [[ ${file_type} = 'png' ]]; then
            convert ${found_file} -resize ${direction}${size} finished/${file_name}_${size_shape}.${file_ext} &> /dev/null
        fi

        if [[ ${file_type} = 'webp' ]]; then
            cwebp ${found_file} -mt -resize ${x} ${y} -o finished/${file_name}_${size_shape}.${file_ext} &> /dev/null
        fi

        rm ${found_file}
    done
```

The `functions.sh` file has a function in it `file_type` that return the self reported `file` type.

The interesting part here is that the file sizes converted to are dynamically calculated by the names of the folders supplied in both the x and y size. No cropping occurs and it will also attempt to resize bigger as it assumes you know what you are doing.

### Finished
The final output end up like this:
```
house_300x218.png
house_551x400.png
house_700x508.png
house.flif
house_300x218.webp
house_551x400.webp
house_700x508.webp
house.png
```

In the resizer there were three folders: 300x, 400y and 700x. Include the `flif` and the original you have all the images you need to make the `<picture>` and `srcset` work.

### Putting it all together
Running these can be done like this (from a central/main script):
```bash
./drop.sh ${1} &
./finished.sh ${2} &

formatter/run.sh &
resizer/run.sh &
renamer/run.sh &
```

`${1}` and `${2}` are the command line arguments for drop folder to watch and output folder respectively. Each child script has to be run as a background process for the aforementioned reason of `inotifywait` blocking the process.

Done.

<aside>
The `picture` tag has one caveat that might be non obvious: while it will skip over file formats that it doesn't recognise, if the tag suggests a format that it does recognise but the file doesn't exist, then a 404 is returned for the whole image and the next format in the tag is not looked at, even if the file were to exist. Be aware.
</aside>
