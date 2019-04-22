Title: Lets make a terrible Markdown extension pt1 - Background
Blurb: Part one of making a Python Markdown extension
Tags: Markdown,Python
Parent: 12

In this page:

[TOC]

##Downloads
You can play along at home by [looking at the full source here](https://assets.themetacity.com/code/theMetaCityMarkdown) and [download a gzipped version here](https://assets.themetacity.com/code/theMetaCityMarkdown.tar.gz).

##Background
Writing blog articles as vanilla HTML is no fun. To that end [Markdown](https://daringfireball.net/projects/markdown/) was created, which for the most part works well enough. Occasionally however there will be some markup that is not processed by the standard Markdown core and is bothersome to type by hand. Custom markup that would be handy to be processed automatically. Lets make some.

There are many variants and parsers in most languages that will take your Markdown and process it into HTML. I am going to do this in Python which has a nice package called Markdown that can process said files.

~~~~{.python}
import Markdown
markdown.markdown('#Example title to parse')
~~~~

This produces the expected output:

~~~~{.html}
<h1>Example title to parse</h1>
~~~~

Riveting.

Thankfully Python Markdown also has a mechanism for extending and adding your own extensions to the standard markup, which we are going to do.

## The problem
Typing out `<video>` tags by hand to be used when replacing gif files is cumbersome and takes an annoyingly long time.

## The solution
Define a new tag that automatically expands to a known good configuration and is quick to type out.

The proposed solution would look like `<gifv baseFilename extension1,extension2,extension3 />`

## The result

~~~{.html}
<gifv gifv-demo-doggos webm />
~~~

becomes

~~~{.html}
<video autoplay="true" class="gifv" controls="false" loop="true">
    <source src="//assets.themetacity.com/gifv/gifv-demo-doggos.webm" />
</video
~~~

and puts a video in like this:

<gifv gifv-demo-doggos webm />

## Assumptions
This is block level tag. Doing this inline does'nt fit with the idea of how I want to use the tag.

Piggybacking on Imgur's marketing with the use of `gifv`.

## Lets crack on
First stop is to [the docs](https://pythonhosted.org/Markdown/extensions/api.html). This is how we are going to define a new tag.

[Step 1.5 is to set up](lets-make-a-terrible-markdown-extension-pt1-5-build-and-deployment) a deployment/build method, which, while optional is pretty handy.

After that is to [dig in and get testing](lets-make-a-terrible-markdown-extension-pt2-getting-testing).