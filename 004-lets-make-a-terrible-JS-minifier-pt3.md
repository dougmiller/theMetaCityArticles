# Lets make a terrible JS minifier: Part 3

Part three and penultimate post of a fun little series on minifying some JavaScript

####################
Type:blog
Tags:JavaScript,Automation
Parent:2
####################

[Following on from part 2][pt2link], I'll show you how I integrate this in to development and deployment workflow for great good!

Throughout the previous examples you would have seen lines like `cat temp.js >> tmcscripts.js` and `rm temp.js`. These actions pretty much sum up how this all works:

1. Copy the first file (searcher.js) to a temp file: `cat searcher.js > temp.js`
2. Run the viable minification on it (part 1)
3. Append (and create since first file) to tmcscripts.js `cat temp > tmcscripts.js`
4. Remove temp file
5. Copy the second file (video.js) to a temp file: `cat video.js > temp.js`
6. Run the viable minification on it (part 1)
7. Append to tmcscripts.js `cat temp >> tmcscripts.js`
8. Remove temp file
9. Repeat for any more scripts to be minified (none ATM)
10. Perform general minification on tmcscripts.js
11. Append any files that need to be there but not (or are already) minified

You will end up with this:

    #!/bin/bash

    # Combine and minify all the js files into one file to save on requests and bytes

    cd media/js # Move to directory with scripts from base directory

    echo 'Begin smooshing searcher.js'

    #  Minify the variable names.
    #  Each script is put in its own function scope so other scripts should (in theory) have no problems with this
    cat searcher.js > temp.js
    grasp -i '#$noResults' -R z temp.js
    ...
    sed -i 's/searchVal\b/n/g' temp.js  # current bug with grasp where it cant parse 3 or more variables on the same line

    # Function names
    grasp -i '#filter' -R m temp.js
    ...

    cat temp.js > myscript.js
    rm temp.js

    echo 'Begin smooshing video.js'

    # Begin the smooshing of video.js
    cat video.js > temp.js

    grasp -i '#videos' -R z temp.js
    ...
    grasp -i '#doc' -R zr temp.js

    cat temp.js >> tmcscripts.js
    rm temp.js

    # Finished with video.js

    echo 'Finished with home brew scripts'
    echo 'Begin general minification'

    sed -i 's/[^:]\/\/.*//g' tmcscripts.js            # Dont need comments (and they become greedy when everything is on a single line). [:] is for urls (which have //)
    ...
    sed -i ':a;N;$!ba;s/\n//g' tmcscripts.js          # New lines N.B. Put this at the end otherwise other operation (like get rid of leading white space) get confused

    echo 'Finished  general minification'

    echo 'Adding in GA (unmolested)'
    # GS is already optimised and mucking with it further seems to break things so just append it at the end
    cat ga.js >> tmcscripts.js

    echo 'Finished with GA'
    echo 'Finished combining all JavaScript files'


Now we have a minified file ready to be deployed and dealt with (which is another post).


[pt2link]: /blog/lets-make-a-terrible-JS-minifier-pt2 "Lets make a terrible JS minifier: Part 2"
