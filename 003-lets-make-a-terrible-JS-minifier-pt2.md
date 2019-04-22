Title: Lets make a terrible JS minifier: Part 2
Blurb: Part two of a fun little series on minifying some JavaScript
Tags: JavaScript,Automation
Parent: 2

[Following on from part 1][pt1link], I am now going to show the next step in minification and then compare results from minified and non minified files.

Now that we have minified variable names, lets look at removing some of the syntax we need to help understand the program by aren't actually necessary to make things work.

Lets go back to searcher and strip things out.

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

Lots of what you see here is not necessary of the program to execute correctly: comments, white space, new lines etc. Lets strip them out and see what we get. sed is our friend here again.

    sed -i 's/[^:]\/\/.*//g' tmcscripts.js                          # Dont need comments (and they become greedy when everything is on a single line). [:] is for urls (which have //)
    sed -i 's/ #\*.*$//'g tmcscripts.js                             # Dont need comments (and they become greedy when everything is on a single line)
    sed -i '/console.*/'d tmcscripts.js                             # Debug statements
    sed -i 's/^[ \t]*//' tmcscripts.js                              # Leading whitespace and tabs N.B in theory there shouldnt need to be tabs anywhere but I am sure there will be
    sed -i 's/[ \t]*$//' tmcscripts.js                              # Trailing whitespace and tabs
    sed -i 's/ () /()/g' tmcscripts.js                              # Spaces in 'function () {'
    sed -i 's/ + /+/g' tmcscripts.js                                # Spaces in 'x + y'
    sed -i 's/ || /||/g' tmcscripts.js                              # Spaces in 'x || y'
    sed -i 's/for (/for(/g' tmcscripts.js                           # Spaces in 'for (' Usually in a for loop
    sed -i 's/; /;/g' tmcscripts.js                                 # Spaces in '; ' Usually in a for loop
    sed -i 's/ \(&lt;\|&lt;=\|&gt;\|&gt;=\) /\1/g' tmcscripts.js    # Spaces in 'x &lt; y|x &gt; y|x &lt;= y|x &gt;= y' Usually in a for loop
    sed -i 's/ \(+=\|-=\) /\1/g' tmcscripts.js                      # Spaces in 'x += y|x -= y'
    sed -i 's/ \(=\+\) /\1/g' tmcscripts.js                         # Spaces in 'x = y', 'x == y', 'x === y'
    sed -i 's/) {/){/g' tmcscripts.js                               # End of parameter list and opening curly brace
    sed -i 's/if (/if(/g' tmcscripts.js                             # End of if and opening round bracket
    sed -i 's/, \(.\)/,\1/g' tmcscripts.js                          # Comma space anything
    sed -i ':a;N;$!ba;s/\n//g' tmcscripts.js                        # New lines N.B. Put this at the end otherwise other operation (like get rid of leading white space) get confused

Which results in this nice guy here:

`$(document).ready(function(){"use strict";var z,y,x,w,v,u,t,s;z=$('#noresults');y=$('#searchinput');x=$('#workshopBlurbEntries');w=null;v=true;u=location;t=history;s=window;function k(){if(t.state !== undefined){t.pushState({"tag": undefined},"theMetaCity - Workshop","/workshop/");}z.hide();x.fadeOut(150,function(){$('header ul li',this).removeClass('searchMatchTag');$('header h1 a span',this).removeClass('searchMatchTitle');$(".workshopentry",this).show();});x.fadeIn(150);}function l(r){if(r===undefined){k();} else {var q=r.replace(/[.?*+^$\[\]\\(){}|]/g,"\\$&"),p=new RegExp('('+q+')','ig');x.fadeOut(150,function(){z.hide();$('header',this).each(function(){$(this).parent().hide();$('li',this).removeClass('searchMatchTag');$('h1',this).each(function(){var o=$('a',this).text();if(o.match(p)){o=o.replace(p,'<span class="searchMatchTitle">$1</span>');$('a',this).html(o);$(this).closest('.workshopentry').show();} else {$('a',this).html(o);}});$('li',this).each(function(){if($(this).text().match(p)){$(this).addClass('searchMatchTag');$(this).closest('.workshopentry').show();}});});if($('.workshopentry[style*="block"]').length===0){z.show();}x.fadeIn(150);});}}$('header ul li a',x).on('click',function(){t.pushState({"tag": $(this).text()},"theMetaCity - Workshop - "+$(this).text(),"/workshop/tag/"+$(this).text());y.val('');l($(this).text());return false;});y.on('keyup',function(){clearTimeout(w);if($(this).val().length){w=setTimeout(function(){var n=y.val();t.pushState({"tag": n},"theMetaCity - Workshop - "+n,"/workshop/tag/"+n);l(n);},500);}if($(this).val().length===0){w=setTimeout(function(){k();},500);}});$('#reset').on('click',function(){y.val('');k();});s.addEventListener("popstate",function (event){if(event.state===null){k();} else {if(event.state.tag !== undefined){y.val(event.state.tag);l(event.state.tag);}}});z.hide();if(v){var locArray=u.pathname.split('/');if(locArray[2]==='tag' && locArray[3] !== undefined){t.pushState({"tag": locArray[3]},"theMetaCity - Workshop - "+locArray[3],"/workshop/tag/"+locArray[3]);l(locArray[3]);} else if(locArray[2]===''){}v=false;}});`

One caveat here is that this is not a general purpose minifier. The files it processes need to conform to certain standard formatting: no multi-line comments (using /\* \*/) and code deliberately designed to confuse these obviously (E.G. `ar foo,       bar`). This is made all the easier for having a single developer on this. hopefully the comments for each line help make sense as to what is going on. The general format is: `sed -i` to apply sed to a file in place (edit the file and don't pipe to output), `s/foo/bar/`, s for substitute (foo with bar) to replace and `g` to do this to the whole file (globally) and not just once. `g` can be replaced with a number (n) so the operation is applied n times. The default when omitting g or n is once.

So where does this get us?
<table>
    <thead>
        <th>file</th>
        <th>presize (b)</th>
        <th>postsize (b)</th>
    </thead>
    <tr>
        <td>searcher.js</td>
        <td>4701</td>
        <td>1940</td>
    </tr>
    <tr>
        <td>video.js</td>
        <td>12575</td>
        <td>5069</td>
    </tr>
    <tr>
        <td>ga.js</td>
        <td>391</td>
        <td>391</td>
    </tr>
    <tr>
        <td>combined</td>
        <td>17667</td>
        <td>7400</td>
    </tr>
</table>

Note the ga.js is Google Analytics, which is already minified and so all we need to do is add it to our tmcscripts.js file and we are good to go. So what we get here is a file reduced to 41% of it's original size and two fewer http requests. Not too bad.

In the next part I will go though automating this as part of the build and deployment.

[pt1link]: /blog/lets-make-a-terrible-JS-minifier "Lets make a terrible JS minifier: Part 1"
