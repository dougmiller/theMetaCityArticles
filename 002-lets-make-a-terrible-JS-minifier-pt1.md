Title: Lets make a terrible JS minifier: Part 1
Blurb: Part one of a fun little series on minifying some JavaScript
Tags: JavaScript,Automation
Parent: 2

Minifying JavaScript is a pretty handy thing to do: it reduces http requests, total download size and so makes peoples experience of your site better.
So lets write a script to do some minifying for us.
First we need some JS to minify: lets use he files running on this site. For this example we will be looking at what we can do to make the files small and how we can combine them to reduce http requests.
The files in question can [be found on GitHub][ghTMC] as the ones on this site are already minified and incomprehensible.
The general approach we are taking today: reducing variable and function names to smallest possible size (saves on bandwidth and the interpretor doesn't care anyway), remove all extraneous formatting (the JS processor does not care about readability) and mash all the files together (saves on https requests).
The order we do these operations to the file can be import and and I will point out where you should look if something were to come up. The first pass I am going to take on the files is a pretty naive one but gets the job done (somewhat) and then we will move on to something a bit more appropriate.
The first file in question is the one that drives the search [on the workshop page (searcher.js)][ghSearcher.js].

    $(document).ready(function () {
        "use strict";
        var $noResults, $searchBox, $entries, searchTimeout, firstRun, loc, hist, win;
        $noResults = $('#noresults');
        $searchBox = $('#searchinput');
        $entries = $('#workshopBlurbEntries');
        searchTimeout = null;
        firstRun = true;
        loc = location;
        hist = history;
        win = window;

        function reset() {
            if (hist.state !== undefined) {  // Avoid infinite loops
                hist.pushState({"tag": undefined}, "theMetaCity - Workshop", "/workshop/");
            }
            $noResults.hide();
            $entries.fadeOut(150, function () {
                $('header ul li', this).removeClass('searchMatchTag');
                $('header h1 a span', this).removeClass('searchMatchTitle');  // The span remains but it is destroyed when filtering using the text() function
                $(".workshopentry", this).show();
            });
            $entries.fadeIn(150);
        }

        function filter(searchTerm) {
            if (searchTerm === undefined) {  // Only history api should push undefined to this, explicitly taken care of otherwise
                reset();
            } else {
                var rePattern = searchTerm.replace(/[.?*+^$\[\]\\(){}|]/g, "\\$&"), searchPattern = new RegExp('(' + rePattern + ')', 'ig');  // The brackets add a capture group

                $entries.fadeOut(150, function () {
                    $noResults.hide();

                    $('header', this).each(function () {
                        $(this).parent().hide();

                        // Clear results of previous search
                        $('li', this).removeClass('searchMatchTag');

                        // Check the title
                        $('h1', this).each(function () {
                            var textToCheck = $('a', this).text();
                            if (textToCheck.match(searchPattern)) {
                                textToCheck = textToCheck.replace(searchPattern, '<span class="searchMatchTitle">$1</span>');  //capture group ($1) used so that the replacement matches the case and you don't get weird capitolisations
                                $('a', this).html(textToCheck);
                                $(this).closest('.workshopentry').show();
                            } else {
                                $('a', this).html(textToCheck);
                            }
                        });

                        // Check the tags
                        $('li', this).each(function () {
                            if ($(this).text().match(searchPattern)) {
                                $(this).addClass('searchMatchTag');
                                $(this).closest('.workshopentry').show();
                            }
                        });
                    });

                    if ($('.workshopentry[style*="block"]').length === 0) {
                        $noResults.show();
                    }

                    $entries.fadeIn(150);
                });
            }
        }

        $('header ul li a', $entries).on('click', function () {
            hist.pushState({"tag": $(this).text()}, "theMetaCity - Workshop - " + $(this).text(), "/workshop/tag/" + $(this).text());
            $searchBox.val('');
            filter($(this).text());
            return false;  // Using the history API so no page reloads/changes
        });

        $searchBox.on('keyup', function () {
            clearTimeout(searchTimeout);
            if ($(this).val().length) {
                searchTimeout = setTimeout(function () {
                    var searchVal = $searchBox.val();
                    hist.pushState({"tag": searchVal}, "theMetaCity - Workshop - " + searchVal, "/workshop/tag/" + searchVal);
                    filter(searchVal);
                }, 500);
            }

            if ($(this).val().length === 0) {
                searchTimeout = setTimeout(function () {
                    reset();
                }, 500);
            }
        });

        $('#reset').on('click', function () {
            $searchBox.val('');
            reset();
        });

        win.addEventListener("popstate", function (event) {
            console.info(hist.state);
            if (event.state === null) { // Start of history chain on this page, direct entry to page handled by firstRun)
                reset();
            } else {
                if (event.state.tag !== undefined) {
                    $searchBox.val(event.state.tag);
                    filter(event.state.tag);
                }
            }
        });

        $noResults.hide();

        if (firstRun) {                               // 0     1     2        3      4 (if / present)
            var locArray = loc.pathname.split('/');   // '/workshop/tag/searchString/
            if (locArray[2] === 'tag' && locArray[3] !== undefined) {    // Check for direct link to tag (i.e. if something in [3] search for it)
                hist.pushState({"tag": locArray[3]}, "theMetaCity - Workshop - " + locArray[3], "/workshop/tag/" + locArray[3]);
                filter(locArray[3]);
            } else if (locArray[2] === '') {   // Root page and really shouldn't do anything
                //hist.pushState({"tag": undefined}, "theMetaCity - Workshop", "/workshop/");
            }   // locArray[2] === somepagenum is an actual page and what should be allowed to happen by itself

            firstRun = false;
            // Save state on first page load
        }
    });

As you can see, a horrible mix of jQuery and vanilla JS and terrible clunky junk with plenty of bugs. But lets start stripping out things we do not need. First up is the variable and function names which can be shortened to individual letters to save on bytes during transmission.

    #  Minify the variable names.
    #  Each script is put in its own function scope so other scripts should (in theory) have no problems with this
    cat searcher.js > temp
    sed -i 's/$noResults\b/z/g' temp
    sed -i 's/$searchBox\b/y/g' temp
    sed -i 's/$entries\b/x/g' temp
    sed -i 's/searchTimeout\b/w/g' temp
    sed -i 's/firstRun\b/v/g' temp
    sed -i 's/loc\b/u/g' temp
    sed -i 's/hist\b/t/g' temp
    sed -i 's/win\b/s/g' temp
    sed -i 's/textToCheck\b/r/g' temp
    sed -i 's/searchPattern\b/q/g' temp
    sed -i 's/address\b/p/g' temp
    sed -i 's/searchString\b/o/g' temp
    sed -i 's/searchTerm\b/n/g' temp

    # Function names
    sed -i 's/filter(/m(/g' temp
    sed -i 's/reset(/l(/g' temp

    # Copy over ready for the next stage of processing
    cat temp > tmcscripts.js

The actual process is pretty straight forward: edit in place (`-i`) the file (in temp for reasons explained later) searching and replacing each instance of the variable and function names. Started at z and worked backwards so as to try and avoid collisions with `i` and other counters (although there are none in this file). Although this works pretty well it is still kind of naive and prone to error. What happens if there is a string and variable string the same name, or get the regex just slightly wrong? A much better solution is to use an abstract syntax tree to parse the file and replace symbols that way. One such exists: [tool for this is graspjs.com][grasp].

    #  Minify the variable names.
    #  Each script is put in its own function scope so other scripts should (in theory) have no problems with this
    cat searcher.js > temp.js
    grasp -i '#$noResults' -R z temp.js
    grasp -i '#$searchBox' -R y temp.js
    grasp -i '#$entries' -R x temp.js
    grasp -i '#searchTimeout' -R w temp.js
    grasp -i '#firstRun' -R v temp.js
    grasp -i '#loc' -R u temp.js
    grasp -i '#hist' -R t temp.js
    grasp -i '#win' -R s temp.js
    grasp -i '#searchTerm' -R r temp.js
    grasp -i '#rePattern' -R q temp.js
    grasp -i '#searchPattern' -R p temp.js
    grasp -i '#textToCheck' -R o temp.js
    sed -i 's/searchVal\b/n/g' temp.js  # current bug with grasp where it cant parse 3 or more variables on the same line

    # Function names
    grasp -i '#filter' -R m temp.js
    grasp -i '#reset' -R l temp.js

    cat temp.js > tmcscripts.js
    rm temp.js

Which gives us:

    $(document).ready(function () {
        "use strict";
        var z, y, x, w, v, u, t, s;
        z = $('#noresults');
        y = $('#searchinput');
        x = $('#workshopBlurbEntries');
        w = null;
        v = true;
        u = location;
        t = history;
        s = window;

        function k() {
            if (t.state !== undefined) {  // Avoid infinite loops
                t.pushState({"tag": undefined}, "theMetaCity - Workshop", "/workshop/");
            }
            z.hide();
            x.fadeOut(150, function () {
                $('header ul li', this).removeClass('searchMatchTag');
                $('header h1 a span', this).removeClass('searchMatchTitle');  // The span remains but it is destroyed when filtering using the text() function
                $(".workshopentry", this).show();
            });
            x.fadeIn(150);
        }

        function l(r) {
            if (r === undefined) {  // Only history api should push undefined to this, explicitly taken care of otherwise
                k();
            } else {
                var q = r.replace(/[.?*+^$\[\]\\(){}|]/g, "\\$&"), p = new RegExp('(' + q + ')', 'ig');  // The brackets add a capture group

                x.fadeOut(150, function () {
                    z.hide();

                    $('header', this).each(function () {
                        $(this).parent().hide();

                        // Clear results of previous search
                        $('li', this).removeClass('searchMatchTag');

                        // Check the title
                        $('h1', this).each(function () {
                            var o = $('a', this).text();
                            if (o.match(p)) {
                                o = o.replace(p, '<span class="searchMatchTitle">$1</span>');  //capture group ($1) used so that the replacement matches the case and you don't get weird capitolisations
                                $('a', this).html(o);
                                $(this).closest('.workshopentry').show();
                            } else {
                                $('a', this).html(o);
                            }
                        });

                        // Check the tags
                        $('li', this).each(function () {
                            if ($(this).text().match(p)) {
                                $(this).addClass('searchMatchTag');
                                $(this).closest('.workshopentry').show();
                            }
                        });
                    });

                    if ($('.workshopentry[style*="block"]').length === 0) {
                        z.show();
                    }

                    x.fadeIn(150);
                });
            }
        }

        $('header ul li a', x).on('click', function () {
            t.pushState({"tag": $(this).text()}, "theMetaCity - Workshop - " + $(this).text(), "/workshop/tag/" + $(this).text());
            y.val('');
            l($(this).text());
            return false;  // Using the history API so no page reloads/changes
        });

        y.on('keyup', function () {
            clearTimeout(w);
            if ($(this).val().length) {
                w = setTimeout(function () {
                    var n = y.val();
                    t.pushState({"tag": n}, "theMetaCity - Workshop - " + n, "/workshop/tag/" + n);
                    l(n);
                }, 500);
            }

            if ($(this).val().length === 0) {
                w = setTimeout(function () {
                    k();
                }, 500);
            }
        });

        $('#reset').on('click', function () {
            y.val('');
            k();
        });

        s.addEventListener("popstate", function (event) {
            console.info(t.state);
            if (event.state === null) { // Start of history chain on this page, direct entry to page handled by firstRun)
                k();
            } else {
                if (event.state.tag !== undefined) {
                    y.val(event.state.tag);
                    l(event.state.tag);
                }
            }
        });

        z.hide();

        if (v) {                               // 0     1     2        3      4 (if / present)
            var locArray = u.pathname.split('/');   // '/workshop/tag/searchString/
            if (locArray[2] === 'tag' && locArray[3] !== undefined) {    // Check for direct link to tag (i.e. if something in [3] search for it)
                t.pushState({"tag": locArray[3]}, "theMetaCity - Workshop - " + locArray[3], "/workshop/tag/" + locArray[3]);
                l(locArray[3]);
            } else if (locArray[2] === '') {   // Root page and really shouldn't do anything
                //hist.pushState({"tag": undefined}, "theMetaCity - Workshop", "/workshop/");
            }   // locArray[2] === somepagenum is an actual page and what should be allowed to happen by itself

            v = false;
            // Save state on first page load
        }
    });

Pretty straightforward to use here and not much difference in logic compared to sed: find a variable/function replace it with a short name. One note you have seen is that it is still pretty new (2 months at the time of writing this) and there are some bugs in there which are easily managed. This is a pretty trivial use with nothing too complex to confuse things. Lets look at a better use: video.js

    $(document).ready(function () {
        "use strict";
        var videos = $("video"), doc = document, fsElement;

        Number.prototype.leftZeroPad = function (numZeros) {
            var n = Math.abs(this),
                zeros = Math.max(0, numZeros - Math.floor(n).toString().length),
                zeroString = Math.pow(10, zeros).toString().substr(1);
            if (this < 0) {
                zeroString = '-' + zeroString;
            }
            return zeroString + n;
        };

        function isVideoPlaying(video) {
            return !(video.paused || video.ended || video.seeking || video.readyState < video.HAVE_FUTURE_DATA);
        }

        // Pass in object of the video to play/pause and the control box associated with it
        function playPause(video) {
            var playPauseButton = $(".playPauseButton", video.parent)[0];
            if (isVideoPlaying(video)) {
                video.pause();
                playPauseButton.src = "/media/site-images/videoicons/smallplay.svg";
            } else {
                video.play();
                playPauseButton.src = "/media/site-images/videoicons/smallpause.svg";
            }
        }

        function rawTimeToFormattedTime(rawTime) {
            var chomped, seconds, minutes;
            chomped = Math.floor(rawTime);
            seconds = chomped % 60;
            minutes = Math.floor(chomped / 60);
            return minutes.leftZeroPad(2) + ":" + seconds.leftZeroPad(2);
        }

        $(videos).each(function () {
            var video = this, $videoContainer, $controlsBox, $playPauseButton, $progressBar, $startPoster, startPoster, $endPoster, customEndPoster, errorPoster, $currentTimeSpan, $durationTimeSpan;

            if (this.controls) {
                this.controls = false;
            }

            $(video).on("timeupdate",function () {
                $progressBar[0].value = (video.currentTime / video.duration) * 1000;
                $currentTimeSpan.text(rawTimeToFormattedTime(video.currentTime));

            }).on("loadedmetadata",function () {
                    var canPlayVid = false;
                    $("source", $(video)).each(function () {
                        if (video.canPlayType($(this).attr("type"))) {
                            canPlayVid = true;
                        }
                    });
                    if (!canPlayVid) {
                        errorPoster = "/media/site-images/movieerror.svg";
                        $.get(errorPoster, function (svg) {
                            errorPoster = doc.importNode(svg.documentElement, true);

                            $(errorPoster).attr("class", "poster errorposter");
                            $(errorPoster).attr("height", $(video).height());
                            $(errorPoster).attr("width", $(video).width());

                            $("source", $(video)).each(function () {
                                var newText = doc.createElementNS("http://www.w3.org/2000/svg", "tspan");
                                var link = doc.createElementNS("http://www.w3.org/2000/svg", "a");
                                newText.setAttributeNS(null, "x", "50%");
                                newText.setAttributeNS(null, "dy", "1.2em");
                                link.setAttributeNS("http://www.w3.org/1999/xlink", "href", this.src);
                                link.appendChild(doc.createTextNode(this.src));
                                newText.appendChild(link);

                                $("#sorrytext", errorPoster).append(newText);
                            });

                            $videoContainer.append(errorPoster);
                            $($videoContainer).trigger("reposition");
                        });
                    } else {
                        $($currentTimeSpan).text(rawTimeToFormattedTime(this.currentTime));
                        $($durationTimeSpan).text(rawTimeToFormattedTime(this.duration));
                    }

                }).on("click",function () {
                    playPause(video);
                }).on("ended",function () {
                    $controlsBox.css({'opacity': 0});

                    // Poster to show at end of movie
                    if (video.dataset.endposter) {
                        customEndPoster = video.dataset.endposter;
                    } else {
                        customEndPoster = "/media/site-images/endofmovie.svg";  // If none supplied, use our own, generic one
                    }
                    // Get the poster and make it inline
                    // File is SVG so usual jQuery rules may not apply
                    // File needs to have at least one element with "playButton" as class
                    $.get(customEndPoster, function (svg) {
                        $endPoster = doc.importNode(svg.documentElement, true);
                        $endPoster = $($endPoster);

                        $endPoster.attr("class", "poster endposter");
                        $endPoster.attr("height", $(video).height());
                        $endPoster.attr("width", $(video).width());

                        $("#playButton", $endPoster).on("click", function () {
                            playPause(video);
                            $endPoster.remove(); // done with poster forever
                        });
                        $videoContainer.append($endPoster);
                        $($videoContainer).trigger("reposition");
                    });
                }).on("play", function () {

                });

            // Setup the div container for the video, controls and poster
            $videoContainer = $(video).wrap(
                $('<div></div>', {
                    class: 'videoContainer'
                }).on("mouseenter",function () {
                        $endPoster = $(".endposter", this); // This is NOT added to the whole script scope so have to rescope it here
                        errorPoster = $(".errorposter", this); // This is NOT added to the whole script scope so have to rescope it here
                        //   Not played yet              Finished playing              Cant play format
                        if ($startPoster.parent().length || $endPoster.parent().length || errorPoster.parent().length) {
                            $controlsBox.css({'opacity': 0});
                        } else {
                            $controlsBox.fadeTo(400, 1);
                            $controlsBox.clearQueue();
                        }
                    }).on("mouseleave",function () {
                        $controlsBox.fadeTo(400, 0);
                        $controlsBox.clearQueue();
                    }).on("reposition", function () {
                        // Move posters and controls back into position after video position updated
                        var videoContainerOffset = $videoContainer.offset(),
                            videoContainerWidth = $videoContainer.width(),
                            heightsTogether = Math.floor(videoContainerOffset.top + $videoContainer.height() - $controlsBox.height()),
                            $endPoster = $(".endposter", this),
                            $errorPoster = $(".errorposter", this);

                        $($startPoster, this).offset({top: videoContainerOffset.top, left: videoContainerOffset.left});

                        $endPoster.offset({top: videoContainerOffset.top, left: videoContainerOffset.left});
                        $endPoster.attr("height", $(video).height());
                        $endPoster.attr("width", $(video).width());

                        $errorPoster.offset({top: videoContainerOffset.top, left: videoContainerOffset.left});

                        $controlsBox.offset({top: heightsTogether, left: videoContainerOffset.left});
                        $controlsBox.width(videoContainerWidth - 2); // 2 is for borders
                    })
            ).parent(); // Return the newly created wrapper div (brand new parent of the video)

            $controlsBox = $("<div></div>", {
                class: "videoControls",
                css: {
                    opacity: 0
                }
            }).appendTo($videoContainer);

            // Setup play/pause button
            $playPauseButton = $("<img />", {
                class: "playPauseButton",
                src: "/media/site-images/videoicons/smallplay.svg"
            }).on("click",function () {
                    playPause(video);
                }).appendTo($controlsBox);

            $durationTimeSpan = $("<span></span>", {
                class: "timespan"
            }).appendTo($controlsBox);

            // Setup progress bar
            $progressBar = $("<input />", {
                type: "range",
                min: 0,
                max: 1000,
                value: 0
            }).on("change",function () {
                    video.currentTime = video.duration * (this.value / 1000);
                }).on("mousedown",function () {
                    video.pause();
                }).on("mouseup",function () {
                    video.play();
                }).appendTo($controlsBox);

            $currentTimeSpan = $("<span></span>", {
                class: "timespan currenttimespan"
            }).appendTo($controlsBox);

            // Full screen
            $("<img />", {
                class: "fullscreenButton",
                src: "/media/site-images/videoicons/fullscreen.svg"
            }).on("click",function () {
                    fsElement = video;
                    if (video.requestFullScreen) {
                        video.requestFullScreen();
                    } else if (video.webkitRequestFullScreen) {
                        video.webkitRequestFullScreen();
                    } else if (video.mozRequestFullScreen) {
                        video.mozRequestFullScreen();
                    }
                }).appendTo($controlsBox);

            // Posters to show before the user plays the video
            startPoster = this.dataset.startposter;
            if (!startPoster) {
                startPoster = "generic";  // If none supplied, use our own, generic one
            }
            // Get the poster and make it inline
            // File is SVG so usual jQuery rules may not apply
            // File needs to have at least one element with "playButton" as class
            $.get("https://assets.themetacity.com/video/" + startPoster + ".startposter.svg", function (svg) {
                $startPoster = doc.importNode(svg.documentElement, true);
                $startPoster = $($startPoster);

                $startPoster.attr("class", "poster");
                $startPoster.attr("height", $(video).height());
                $startPoster.attr("width", $(video).width());

                $("#playButton", $startPoster).on("click", function () {
                    video.load();   // Initial data and metadata load events may have fired before they can be captured so manually fire them
                    playPause(video);
                    $startPoster.remove(); // done with poster forever
                });
                $videoContainer.append($startPoster);
                $($videoContainer).trigger("reposition");
            });

            // Add whe whole lot onto the page
            $videoContainer.append($controlsBox);

            $($videoContainer).trigger("reposition"); //Get its position right.
        });

        // Handle coming out of fullscreen
        $(doc).on("webkitfullscreenchange mozfullscreenchange fullscreenchange", function () {
            var isFullScreen = doc.fullScreen || doc.mozFullScreen || doc.webkitIsFullScreen;

            $(fsElement).each(function () {  // set to script scope as fullScreenElement appears to not work (yet?)
                var video = this, videoTime = video.currentTime;
                if (isFullScreen) {
                    $("source", video).each(function () {
                        // .dataset.fullscreen is is treated a boolean, but it is just truthy string
                        // This function uses a standard format of names of full screen appropriate vids as shown below:
                        // original: originalvid.xyz            full screen: originalvid.fullscreen.xyz
                        // N.B. Can not have period (".") in original file same except for filetype
                        if (this.dataset.fullscreen) {
                            var splitSrc = this.src.split(".");
                            this.src = splitSrc[0] + "." + splitSrc[1] + "." + splitSrc[2] + ".fullscreen." + splitSrc[3];
                        }
                        video.load();
                    });
                } else {  // Have left fullscreen and need to return to lower res video
                    $("source", video).each(function () {
                        // Remove the full screen and go back to the original file
                        if (this.dataset.fullscreen) {
                            var splitSrc = this.src.split(".");
                            this.src = splitSrc[0] + "." + splitSrc[1] + "." + splitSrc[2] + "." + splitSrc[4];
                        } // Nothing was changed if data-fullscreen is false so no need to do anything

                        $(this).parent().load();  // The video
                        $(this).parent().trigger("reposition");  // The video container box
                    });
                }
                $(video).on("loadedmetadata", function () {
                    this.currentTime = videoTime;  // Skip to the time before we went full screen
                    playPause(video);
                });
            });
        });
        $(window).on("resize", function () {
            $(videos).each(function () {
                $(this).parent().trigger("reposition");
            });
        });
    });

Bit more complex and a bit more involved. Set up with the usual sed treatment with one notable wrinkle.

    cat video.js > temp.js

    sed -i 's/videos\b/z/g' temp.js
    sed -i 's/video\b/y/g' temp.js
    sed -i 's/"y"/"video"/1' temp.js  # Undo the first (and only) "video" instance
    sed -i 's/playPauseButton/x/g' temp.js
    sed -i 's/vidIndex/w/g' temp.js
    ...

The second sed line highlights the problem here as it can not differentiate between `video.blah()` and `var videos = $("video");`. This is pretty common (and is one of the use cases on the graspjs website). The sed solution the the sed problem is to go in again and turn back the variable: `sed -i 's/"y"/"video"/1' temp.js  # Undo the first (and only) "video" instance`. Not very wieldy.

Lets do better:

    # Begin the smooshing of video.js
    cat video.js > temp.js

    grasp -i '#videos' -R z temp.js
    grasp -i '#leftZeroPad' -R y temp.js
    grasp -i '#numZeros' -R x temp.js
    grasp -i '#zeros' -R w temp.js
    grasp -i '#zeroString' -R v temp.js
    grasp -i '#isVideoPlaying' -R u temp.js
    grasp -i '#video' -R t temp.js
    grasp -i '#playPause' -R s temp.js
    grasp -i '#vid' -R r temp.js
    grasp -i '#playPauseButton' -R q temp.js
    grasp -i '#rawTimeToFormattedTime' -R p temp.js
    grasp -i '#rawTime' -R o temp.js
    grasp -i '#chomped' -R n temp.js
    grasp -i '#seconds' -R m temp.js
    grasp -i '#minutes' -R l temp.js
    grasp -i '#$video' -R k temp.js
    grasp -i '#$videoContainer' -R j temp.js
    grasp -i '#$controlsBox' -R i temp.js
    grasp -i '#$playPauseButton' -R h temp.js
    grasp -i '#$progressBar' -R g temp.js
    grasp -i '#$poster' -R f temp.js
    grasp -i '#customPoster' -R e temp.js
    grasp -i '#$endPoster' -R d temp.js
    grasp -i '#customEndPoster' -R c temp.js
    grasp -i '#$errorPoster' -R b temp.js
    grasp -i '#$currentTimeSpan' -R a temp.js
    grasp -i '#$durationTimeSpan' -R zz temp.js
    grasp -i '#svg' -R zy temp.js
    grasp -i '#canPlayVid' -R zx temp.js
    grasp -i '#newText' -R zw temp.js
    grasp -i '#link' -R zv temp.js
    grasp -i '#videoContainerOffset' -R zu temp.js
    grasp -i '#videoContainerWidth' -R zt temp.js
    grasp -i '#heightsTogether' -R zs temp.js
    grasp -i '#doc' -R zr temp.js

This works out to:
    $(document).ready(function () {
        "use strict";
        var z = $("video"), zr = document, zq;

        Number.prototype.y = function (x) {
            var n = Math.abs(this),
                w = Math.max(0, x - Math.floor(n).toString().length),
                v = Math.pow(10, w).toString().substr(1);
            if (this < 0) {
                v = '-' + v;
            }
            return v + n;
        };

        function u(t) {
            return !(t.paused || t.ended || t.seeking || t.readyState < t.HAVE_FUTURE_DATA);
        }

        // Pass in object of the video to play/pause and the control box associated with it
        function s(t) {
            var q = $(".playPauseButton", t.parent)[0];
            if (u(t)) {
                t.pause();
                q.src = "/media/site-images/videoicons/smallplay.svg";
            } else {
                t.play();
                q.src = "/media/site-images/videoicons/smallpause.svg";
            }
        }

        function p(o) {
            var n, m, l;
            n = Math.floor(o);
            m = n % 60;
            l = Math.floor(n / 60);
            return l.y(2) + ":" + m.y(2);
        }

        $(z).each(function () {
            var t = this, j, i, h, g, $startPoster, startPoster, d, c, b, a, zz;

            if (this.controls) {
                this.controls = false;
            }

            $(t).on("timeupdate",function () {
                g[0].value = (t.currentTime / t.duration) * 1000;
                a.text(p(t.currentTime));

            }).on("loadedmetadata",function () {
                    var zx = false;
                    $("source", $(t)).each(function () {
                        if (t.canPlayType($(this).attr("type"))) {
                            zx = true;
                        }
                    });
                    if (!zx) {
                        b = "/media/site-images/movieerror.svg";
                        $.get(b, function (zy) {
                            b = zr.importNode(zy.documentElement, true);

                            $(b).attr("class", "poster errorposter");
                            $(b).attr("height", $(t).height());
                            $(b).attr("width", $(t).width());

                            $("source", $(t)).each(function () {
                                var zw = zr.createElementNS("http://www.w3.org/2000/svg", "tspan");
                                var zv = zr.createElementNS("http://www.w3.org/2000/svg", "a");
                                zw.setAttributeNS(null, "x", "50%");
                                zw.setAttributeNS(null, "dy", "1.2em");
                                zv.setAttributeNS("http://www.w3.org/1999/xlink", "href", this.src);
                                zv.appendChild(zr.createTextNode(this.src));
                                zw.appendChild(zv);

                                $("#sorrytext", b).append(zw);
                            });

                            j.append(b);
                            $(j).trigger("reposition");
                        });
                    } else {
                        $(a).text(p(this.currentTime));
                        $(zz).text(p(this.duration));
                    }

                }).on("click",function () {
                    s(t);
                }).on("ended",function () {
                    i.css({'opacity': 0});

                    // Poster to show at end of movie
                    if (t.dataset.endposter) {
                        c = t.dataset.endposter;
                    } else {
                        c = "/media/site-images/endofmovie.svg";  // If none supplied, use our own, generic one
                    }
                    // Get the poster and make it inline
                    // File is SVG so usual jQuery rules may not apply
                    // File needs to have at least one element with "playButton" as class
                    $.get(c, function (zy) {
                        d = zr.importNode(zy.documentElement, true);
                        d = $(d);

                        d.attr("class", "poster endposter");
                        d.attr("height", $(t).height());
                        d.attr("width", $(t).width());

                        $("#playButton", d).on("click", function () {
                            s(t);
                            d.remove(); // done with poster forever
                        });
                        j.append(d);
                        $(j).trigger("reposition");
                    });
                }).on("play", function () {

                });

            // Setup the div container for the video, controls and poster
            j = $(t).wrap(
                $('<div></div>', {
                    class: 'videoContainer'
                }).on("mouseenter",function () {
                        d = $(".endposter", this); // This is NOT added to the whole script scope so have to rescope it here
                        b = $(".errorposter", this); // This is NOT added to the whole script scope so have to rescope it here
                        //   Not played yet              Finished playing              Cant play format
                        if ($startPoster.parent().length || d.parent().length || b.parent().length) {
                            i.css({'opacity': 0});
                        } else {
                            i.fadeTo(400, 1);
                            i.clearQueue();
                        }
                    }).on("mouseleave",function () {
                        i.fadeTo(400, 0);
                        i.clearQueue();
                    }).on("reposition", function () {
                        // Move posters and controls back into position after video position updated
                        var zu = j.offset(),
                            zt = j.width(),
                            zs = Math.floor(zu.top + j.height() - i.height()),
                            d = $(".endposter", this),
                            $errorPoster = $(".errorposter", this);

                        $($startPoster, this).offset({top: zu.top, left: zu.left});

                        d.offset({top: zu.top, left: zu.left});
                        d.attr("height", $(t).height());
                        d.attr("width", $(t).width());

                        $errorPoster.offset({top: zu.top, left: zu.left});

                        i.offset({top: zs, left: zu.left});
                        i.width(zt - 2); // 2 is for borders
                    })
            ).parent(); // Return the newly created wrapper div (brand new parent of the video)

            i = $("<div></div>", {
                class: "videoControls",
                css: {
                    opacity: 0
                }
            }).appendTo(j);

            // Setup play/pause button
            h = $("<img />", {
                class: "playPauseButton",
                src: "/media/site-images/videoicons/smallplay.svg"
            }).on("click",function () {
                    s(t);
                }).appendTo(i);

            zz = $("<span></span>", {
                class: "timespan"
            }).appendTo(i);

            // Setup progress bar
            g = $("<input />", {
                type: "range",
                min: 0,
                max: 1000,
                value: 0
            }).on("change",function () {
                    t.currentTime = t.duration * (this.value / 1000);
                }).on("mousedown",function () {
                    t.pause();
                }).on("mouseup",function () {
                    t.play();
                }).appendTo(i);

            a = $("<span></span>", {
                class: "timespan currenttimespan"
            }).appendTo(i);

            // Full screen
            $("<img />", {
                class: "fullscreenButton",
                src: "/media/site-images/videoicons/fullscreen.svg"
            }).on("click",function () {
                    zq = t;
                    if (t.requestFullScreen) {
                        t.requestFullScreen();
                    } else if (t.webkitRequestFullScreen) {
                        t.webkitRequestFullScreen();
                    } else if (t.mozRequestFullScreen) {
                        t.mozRequestFullScreen();
                    }
                }).appendTo(i);

            // Posters to show before the user plays the video
            startPoster = this.dataset.startposter;
            if (!startPoster) {
                startPoster = "generic";  // If none supplied, use our own, generic one
            }
            // Get the poster and make it inline
            // File is SVG so usual jQuery rules may not apply
            // File needs to have at least one element with "playButton" as class
            $.get("https://assets.themetacity.com/video/" + startPoster + ".startposter.svg", function (zy) {
                $startPoster = zr.importNode(zy.documentElement, true);
                $startPoster = $($startPoster);

                $startPoster.attr("class", "poster");
                $startPoster.attr("height", $(t).height());
                $startPoster.attr("width", $(t).width());

                $("#playButton", $startPoster).on("click", function () {
                    t.load();   // Initial data and metadata load events may have fired before they can be captured so manually fire them
                    s(t);
                    $startPoster.remove(); // done with poster forever
                });
                j.append($startPoster);
                $(j).trigger("reposition");
            });

            // Add whe whole lot onto the page
            j.append(i);

            $(j).trigger("reposition"); //Get its position right.
        });

        // Handle coming out of fullscreen
        $(zr).on("webkitfullscreenchange mozfullscreenchange fullscreenchange", function () {
            var zp = zr.fullScreen || zr.mozFullScreen || zr.webkitIsFullScreen;

            $(zq).each(function () {  // set to script scope as fullScreenElement appears to not work (yet?)
                var t = this, videoTime = t.currentTime;
                if (zp) {
                    $("source", t).each(function () {
                        // .dataset.fullscreen is is treated a boolean, but it is just truthy string
                        // This function uses a standard format of names of full screen appropriate vids as shown below:
                        // original: originalvid.xyz            full screen: originalvid.fullscreen.xyz
                        // N.B. Can not have period (".") in original file same except for filetype
                        if (this.dataset.fullscreen) {
                            var zo = this.src.split(".");
                            this.src = zo[0] + "." + zo[1] + "." + zo[2] + ".fullscreen." + zo[3];
                        }
                        t.load();
                    });
                } else {  // Have left fullscreen and need to return to lower res video
                    $("source", t).each(function () {
                        // Remove the full screen and go back to the original file
                        if (this.dataset.fullscreen) {
                            var zo = this.src.split(".");
                            this.src = zo[0] + "." + zo[1] + "." + zo[2] + "." + zo[4];
                        } // Nothing was changed if data-fullscreen is false so no need to do anything

                        $(this).parent().load();  // The video
                        $(this).parent().trigger("reposition");  // The video container box
                    });
                }
                $(t).on("loadedmetadata", function () {
                    this.currentTime = videoTime;  // Skip to the time before we went full screen
                    s(t);
                });
            });
        });
        $(window).on("resize", function () {
            $(z).each(function () {
                $(this).parent().trigger("reposition");
            });
        });
    });

Much better.
[Next time I will go through][ptlink] the next steps in minification and compare end results.


[ghTMC] https://github.com/dougmiller/theMetaCity/tree/master/media/js "See 'theMetaCity' on GitHub."
[ghSearcher.js] https://github.com/dougmiller/theMetaCity/blob/master/media/js/searcher.js "See the file 'searcher.js' on GitHub."
[grasp] http://www.graspjs.com "Grasp homepage"
[pt2link]: /blog/lets-make-a-terrible-JS-minifier-pt2 "Lets make a terrible JS minier: Part 2"