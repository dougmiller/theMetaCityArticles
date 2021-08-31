Title: 15 minute chains and local storage
URL: 15-minute-chains-and-local-storage
Blurb: Test of the local storage API in modern browsers with habit building to boot
Tags: JavaScript,InfoVis

As an exercise in learning some more JavaScript, I decided to write a little goal progress monitor/tracker.

The initial idea is pretty straight forward: a checkbox that shows you if you have checked off the task for today and also shows the past week (or whatever) below it, forming a continuous chain. The idea is to not break the chain (of days completed).

![Initial paper drawing of mockup of design. It shows a check-list of ideas and technologies to used.][initialdesign]

The initial HTML is pretty straightforward with some divs representing each goal/activity to track, and some child divs representing how long you want to track. The H1 is both the name of the activity, and the key used to track it. We will see more on that below.

```
:::html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>15 Minutes Chain</title>
        <meta charset="utf-8">
        <link href="style.css" rel="stylesheet" type="text/css" media="screen"/>
    </head>
    <body>
        <div id="maincontainer">
            <div class="chainContainer">
                <h1>Stretching</h1>
                <div class="today"></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
            </div>
            <div class="chainContainer">
                <h1>Piano</h1>
                <div class="today"></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
            </div>
            <div class="chainContainer">
                <h1>Guitar</h1>
                <div class="today"></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
            </div>
            <div class="chainContainer">
                <h1>Deutsch</h1>
                <div class="today"></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
                <div></div>
            </div>
        </div>
        <h5 id="reset">Reset</h5>
        <script src="script.js" type="application/javascript;version=1.7"></script>
    </body>
</html>
```

The CSS is pretty straightforward too:

```
:::css
@charset "utf-8";

html {
    color: #BCBCBC;
}

body {
    margin: 100px auto auto;
    width: 800px;
}

.chainContainer {
    width: 25%;
    float: left;
    text-align: center;
}

.chainLink {
    height: 50px;
    width: 50px;
    margin-top: 20px;
    margin-left: auto;
    margin-right: auto;
}

.today {
    border: solid gray;
    -moz-box-sizing: border-box;
}

.done {
    background-color: green;
}

.notdone {
    background-color: red;
}
```

You will notice that there are a few Firefox specific things going on. My primary browser is Firefox, so I made this for that. The <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing">first is the -moz-box-sizing</a> in the CSS mostly to make all the boxes appear the same when there is a border and when there is not without messing with border and widths.

The second is the use of `<let>`. This is a JavaScript 1.7 change that I used because I wanted to see how it would affect things. To make it work (at the time of writing) you need to have the <code>type="application/javascript;version=1.7"</code> in your script tag.

The interesting part below is the use of `localStorage` to save the state of your progress. localStorage is a way to assign key:value store bound to the <a href="http://www.whatwg.org/specs/web-apps/current-work/multipage/origin-0.html">origin</a> with 5mb to play with.

```
:::javascript
(function () {
    "use strict";

    function compareDates(date1, date2) {
    console.log(date1 - date2);
        return (date1 - date2) / 86400000;
    }

    let jobs = document.getElementsByClassName("chainContainer");
    let todayDate = new Date();
    todayDate.setHours(0, 0, 0, 0);
    todayDate.setDate(todayDate.getDate());

    //  The chainContainer is setup to have an <h1> as the first element ([0]).
    //  This element's text act's as the local storage key.
    //  I.E. Changing the text will lose your history.
    //  You can have as many of them as you want (fit on a page). Just copy a different one and change the text etc.
    //  Chain links start at [1] and go for as many as you want. 1 per day though. Unless you want to change that.
    let jobHistories = [];
    for (let i = 0; i < jobs.length; i += 1) {
        jobHistories[i] = localStorage.getItem(jobs[i].children[0].textContent);
        
        for (let j = 1; j < jobs[i].children.length; j += 1) {
            jobs[i].children[j].classList.add("chainLink");
        }
    }


    //  Click on 'today'
    for (let i = 0; i < jobs.length; i += 1) {
        jobs[i].children[1].addEventListener("click", function () {
            // Serialise state to local storage
            let state = localStorage.getItem(this.parentNode.children[0].textContent);
            this.classList.add("done");

            if (state === null) {  //  Nothing in local history yet
                state = [];
            } else {
                state = state.split(",");
            }

            if (state.indexOf(todayDate.toDateString()) === -1) {
                state.push(todayDate.toDateString());
                localStorage.setItem(this.parentNode.children[0].textContent, state);
            }
        });
    }

    //  Restore from state
    for (let i = 0; i < jobs.length; i += 1) {
        let prevState;
        try {
            prevState = localStorage.getItem(jobs[i].children[0].textContent).split(",");
            for (let j = 0; j < prevState.length; j += 1) {
                let stateDate = new Date(prevState[j]);
                let dateDiff = compareDates(todayDate, stateDate);

                if (dateDiff > jobs[i].children.length - 2) {
                    break;  // Array out of bounds
                }

                jobs[i].children[dateDiff + 1].classList.add("done");
            }

            for (let j = 2; j < jobs[i].children.length; j += 1) {
                if (!jobs[i].children[j].classList.contains("done")) {
                    jobs[i].children[j].classList.add("notdone");
                }
            }
        } catch (TypeError) { //  No history yet
            for (let j = 2; j < jobs[i].children.length; j += 1) {
                jobs[i].children[j].classList.add("notdone");
            }
        }
    }

    //  Reset the history
    let reset = document.getElementById("reset");
    reset.addEventListener("click", function () {
        for (let i = 0; i < jobs.length; i += 1) {
            localStorage.clear(jobs[i].children[0].textContent);
            for (let j = 1; j < jobs[i].children.length; j += 1) {
                jobs[i].children[j].classList.remove("done");
                jobs[i].children[j].classList.remove("notdone");
            }
            for (let j = 2; j < jobs[i].children.length; j += 1) {
                jobs[i].children[j].classList.add("notdone");
            }
        }
    });
}());
```

First interesting thing here is the use of the `<H1>` as the key part of the key:value pair. This has one nice side effect: adding and removing goals and changing them is straightforward and cheap. Just edit the HTML to add a new `chainContainer` or change the H1 of an existing one. Changing an existing one will make a new key but will not delete the previous one. That means if you change it back, the history come back with it. These keys are still at the users control though, and a history wipe will take them with it.

The next thing to look at is the values themselves. The store is key:value strings. So pushing in objects will convert them to strings. There are a couple of implications of this: there are ONLY strings and not boolean or date objects, and you need to manually de-serialise objects.

Manual de-serialisation is quite straightforward: JavaScript will generally call the `.toString()` method on whatever you push. In this case it is an array which is comma delimited. The de-serialisation is to split it on that comma, and you have strings of what was in your array. Great. Next step (in this case) is to change the strings back to date objects by calling `new Date(string);`. One gotcha here is to make sure that today's date ignores the hours and minutes component. Otherwise, you end up comparing fractions of days with whole days, and you get the wrong answer even though it looks right.

The lot of this [can be found on GitHub here][GitHub].

P.S. I am well aware of the inefficiencies of context switching.

[initialdesign]: https://assets.themetacity.com/image/blog/15minchaininitialdesign.jpg "Inital mockup of design and a checklist of how to go about building it."

[GitHub]: /github "Link to this project on GitHub"