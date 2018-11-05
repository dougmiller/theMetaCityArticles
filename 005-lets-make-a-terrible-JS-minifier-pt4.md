# Lets make a terrible JS minifier: Part 4

This is the fourth and final part of the minifier write up. See [part one][pt1link], [part two][pt2link], [part three][pt3link] or [the lot in the archive][tmcarchive].

It is important to look and the goal of a minifier and how it is produced as it highlights a lot of the processes that goes into software development. The main lesson to take away from this is that any project, process or program is subject to constraints and goals outside of the program itself and you need to be aware of and reflect upon them.

For example: the minifier's goal is to reduce the bandwidth and requests needed to download the JavaScript files and to this end it does a pretty good job. But it could be better. Looking over the resulting code there are a number of subtle ways to make it better: we could optimise for variable replacement symbol size, i.e replacing a 5 byte word with a one byte word is better than replaceing a three byte word with a two byte word). One way to make this more efficient would be to count the lengh of each word and multiply it by the number of times each appears and then replace them in a decending maner which would make sure that the most used words have the shrortest replacements.

There are more ways too: selectors that reference CSS classes are not replaced and some of them are quite wordy which could be a few more bytes saved. What about redundant lines? An entire line of code gone is a potentially much better reduction than a word or two. It is a very deep rabbit hole.

Which brings me back to my point. Any decision about how this works or goal you have for it is subject to trade offs in lots of different areas. By working out length by times used, you need a way to be abole to do that: parse it of some kind of hacked up counting scheme, all of which need to be written and maintained and understood by everyone inviolved in the project. All for a few bytes? It might be worth it but you certainly dont get it for free. Same problem with the CSS selectors too: by minifiying them you reduce readability in both languages so now everything is harder to  understand, maintain and extend. Why not automate that? Sure but that is another dependancy or system to maintain and support. If you remove that one line of code, does it make it harder for your you and others to understand what was is intended and what is happening? What about when you need to make changes? Can you find where the bug is with less code?

The point here is there is no one right answer to this. There is however a finite number of hours in the day, wasted time on mis or unclear communication and potentailly lower hanging fruit to grasp for. Make sure you understand the bigger picture of how your project/system/whatever fits in and where your time is best spent.

This concludes the write up on the minifier. You can see the [whole lot on the archive][tmcarchive].

[pt1link]: /blog/lets-make-a-terrible-JS-minifier-pt1 "Lets make a terrible JS minifier: Part 1"

[pt2link]: /blog/lets-make-a-terrible-JS-minifier-pt2 "Lets make a terrible JS minifier: Part 2"

[pt3link]: /blog/lets-make-a-terrible-JS-minifier-pt3 "Lets make a terrible JS minifier: Part 3"

[tmcarchive]: /blog/archive "theMetaCity archive"
