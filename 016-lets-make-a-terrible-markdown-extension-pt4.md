Title: Lets make a terrible Markdown extension pt4 - Getting coding pt2
URL: lets-make-a-terrible-markdown-extension-pt4-getting-coding-pt2
Blurb: Fifth and final part of making a Python Markdown extension
Tags: Markdown,Python
Parent: 12

In this page:

[TOC]

We now have all the parts in place to write the class that does the actual work of transforming text. As previously mentioned, there are several places throughout the markdown pipeline that we can insert our extension. [Lets read the docs](https://pythonhosted.org/Markdown/extensions/api.html) then have a brief look at each.

##Processors
The general flow here is to look at the source, attempt to parse it and build a tree out of it, making manipulations along the way, then serialising the tree out as HTML.

###Preprocessors
When `markdown` runs, the first process it runs makes the entire source available as a raw string. This will allow you to go through and correct any issues you find or work on the raw strings in some way. It is not smart in any way.

###Block parser
This looks at blocks of text separated by blank lines and attempts to build a `Tree` out of them. It is possible to manipulate this parsing.

###Treeprocessor
After block parsing, the process has built the source into an `ElementTree`. This will let you walk tree and modify it as you need.

###Inline patterns
The next process is to process inline strings i.e., the bold and underline and URL processing and other tags used within a string. It will not process HTMLesque tags.

###Post processor
At this point the `Tree` is serialised to a string and returned. If you need to you can run a `PostProcessor` to work with the output string.

##Example time!

Let's build a preprocessor as we want to make a new tag that allows building of `<video>` tags based on the `<gif>` tag mentioned previously.

From `GifVPreprocessor()` line mentioned previously, lets build the class:

~~~{.python hl_lines="2"}
def extendMarkdown(self, md, md_globals):
    md.preprocessors.add('gifv', GifVPreprocessor(self), '_begin')
~~~

Preprocessors need to inherit from `markdown.preprocessors.Preprocessor` and define one method `run(lines)` with an argument of `lines` which is the entire source document.

~~~{.python hl_lines="6"}
class GifVPreprocessor(Preprocessor):
    def __init__(self, gifv, **kwargs):
        self.gifv = gifv
        super().__init__(**kwargs)

    def run(self, lines):
        pass
~~~

First step is to define a regex to match against when going through each line:

~~~{.python hl_lines="4"}
class GifVPreprocessor(Preprocessor):
    def __init__(self, gifv, **kwargs):
        self.gifv = gifv
        self.RE = re.compile(r'<gifv ([\w0-9_-]+) ([\w0-9_-]+[,?[\w0-9_-]+]?) ?/>$')
        super().__init__(**kwargs)

    def run(self, lines):
        pass
~~~

This will match `<gifv word extension1,extension2,extensionX />` with an optional space at the end there.

The run method is passed the entire source document; it is up to us to deal with it how we want.

~~~{.python hl_lines="9 11"}
class GifVPreprocessor(Preprocessor):
    def __init__(self, gifv, **kwargs):
        self.gifv = gifv
        self.RE = re.compile(r'<gifv ([\w0-9_-]+) ([\w0-9_-]+[,?[\w0-9_-]+]?) ?/>$')
        super().__init__(**kwargs)

    def run(self, lines):
        new_lines = []
        for line in lines:
            m = self.RE.match(line)
            if m:
                # you got a match, do what you ned to
            else:
                new_lines.append(line)  # pass through unmolested
        return new_lines  # sends back the completed source
~~~

Now it is a straightforward matter of breaking out the groups from the regex and building the `<video>` element.
 
~~~{.python}
class GifVPreprocessor(Preprocessor):
    def __init__(self, gifv, **kwargs):
        self.gifv = gifv
           super().__init__(**kwargs)

    def run(self, lines):
        new_lines = []
        for line in lines:
            m = self.RE.match(line)
            if m:
                filename = m.group(1)
                extensions = m.group(2).split(',')
                video = etree.Element('video')
                video.set("autoplay", "true")
                video.set("controls", "false")
                video.set("loop", "true")
                video.set("class", self.gifv.getConfig('css_class'))

                for extension in extensions:
                    source = etree.SubElement(video, "source")
                    source.set('src', '{}{}.{}'.format(self.gifv.getConfig('video_url_base'), filename, extension))

                new_lines.append(etree.tostring(video, encoding="unicode"))
            else:
                new_lines.append(line)  # pass through unmolested
        return new_lines  # sends back the completed source
~~~

Why are we building this as an `ElementTree` and not raw strings? Well you can but raw strings are still a pain to deal with.

Why the `encoding="unicode"`? The etree will attempt to stringify but run into an error where strings are represented as bytes but are expecting strings because of the different ways Python 2 and 3 represent strings.

So you are done. Run the tests and see that it process the tag into what you need.
 
[You can see the full source here](https://assets.themetacity.com/code/theMetaCityMarkdown) and [download a gzipped version here](https://assets.themetacity.com/code/theMetaCityMarkdown.tar.gz).