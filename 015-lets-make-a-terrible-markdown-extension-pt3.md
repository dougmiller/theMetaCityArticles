Title: Lets make a terrible Markdown extension pt3 - Getting coding
Blurb: Fourth part of making a Python Markdown extension
Tags:Markdown,Python
Parent:12

In this page:

[TOC]

Now that we are setup to build and test the code, lets get to coding.

~~~{.python hl_lines="1 9"}
class GifV(Extension):
    def __init__(self, *args, **kwargs):
        self.config = {
            'video_url_base': ['//assets.themetacity.com/gifv/', 'URL of the directory the file resides in'],
            'css_class': ['gifv', 'CSS class to append to the video to identify it as a gifv']
        }
        super().__init__(*args, **kwargs)

    def extendMarkdown(self, md, md_globals):
        md.preprocessors.add('gifv', GifVPreprocessor(self), '_begin')
~~~

Your class definition needs to extent `Extension` which will hook it into the markdown system. This class will also need to define a method `extendMarkdown`.

The init sets up the config options you can define for the extension. These can be overwritten at the time the extension is processed like so:

~~~{.python}
import Markdown
markdown.markdown('Demo text', extensions=[GifV(css_class='other_classname')])
~~~

`extendMarkdown` is where the extension is registered in the markdown process. If your extension needs to do more than one thing, it is just a matter or registering both classes in here.

~~~
def extendMarkdown(self, md, md_globals):
    md.*.add('gifv', ClassWhereWorkHappens(self), '_begin')
    md.*.add('gify', DifferentClassWhereWorkHappens(self), '_begin')
~~~

`md.*.add()` registers the the class with the `mardown` process. The `*` has a few different option depending on the type of extension needed. See further in the guide.

The `_being` string at the end there instructs the `markdown` package as to the order which to run the extension. The order matters as some transformations will affect how others work.

To see the order of the added processors it is a straightforward matter to query the `OrdereredDict` they are stored in, `md.preprocessors`. The usual rules for adding them in are the same as any other `OrdereredDict`.

Next up is to get on with the `GifPreprocessor` class.