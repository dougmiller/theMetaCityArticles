# Lets make a terrible Markdown extension pt1.5 - Build and deployment

##Other pages in this series
 - [Pt. 1 Background](/blog/lets-make-a-terrible-markdown-extension-pt1-background) 
 - Pt. 1.5 Build tools (this page)
 - [P2. 2 Getting testing](/blog/lets-make-a-terrible-markdown-extension-pt2-getting-testing)
 - [Pt. 3 Getting coding](/blog/lets-make-a-terrible-markdown-extension-pt3-getting-coding)
 - [Pt. 4 Getting coding pt2](/blog/lets-make-a-terrible-markdown-extension-pt4-getting-coding-pt2)

In this page:

[TOC]

Building and deploying extensions works well as a package. Here is how to do it in a reasonable way for the extension we are writing.

##Assumptions
You are using a `virtulenv` that is specific to this project. Amongst all the the usual parts it brings `pip` which we will use to do the actual deploying. While `virtualenv` is not needed but it does keep this process much easier to keep straight. How and where you setup `virtualenv` is left as an exercise to the reader. Unless there is a compelling reason mine are usually stored in a dedicated directory `~/.virtualenvs` so as not to pollute the build or working directory.
 
Oh, and unix.
 
##setup.py
This is the config `pip` uses when building and deploying. Looks something like this:

~~~~{.python}
#!/usr/bin/env python

from setuptools import setup

setup(
    name='tmcmarkdown',
    packages=['tmcmarkdown', 'tmcmarkdown.extensions', 'tmcmarkdown.tests'],
    version='1.0.2',
    maintainer="Doug Miller",
    maintainer_email="dougmiller@themetacity.com",
    url="themetacity.com",
    py_modules=[
        'gifv',
    ],
    license='LICENCE.md',
    description='A collection of markdown extensions used on theMetaCity.com',
    long_description=open('./README.txt', 'r').read(),
    install_requires=['markdown']
)
~~~~

<aside>
This is actaully a really good example of why it is advisable to use the `/usr/bin/env` construct. If you haven't see that before, the idea is that rather than hardcoding the path to the executable you want to run the script, you defer to the OS to tell you what the path to the executable is.

This allows you to run the same script under many different environments (i.e. a system installation and a virtualenv installation).

It is a handy way to remove one portability issue which costs nothing in implement.
</aside>

##Installation
The main driver here is obviously `setuptools`. This supersedes `distutils` and comes with `python` >= 3.4

If you have not already, [go read the docs](https://packaging.python.org).

Defining the package here allows up to do two things that are quite useful: install the package we are going to build into the current python environment site-packages and link the package into the current site-packages.

The first idea is useful for when the package is ready to be installed. The second is more interesting as it allows you to install the package via symlink into the site-package which removes the need to run the installation script everytime the package is changed (i.e. during development).

For the first: `python setup.py install` and the second `python setup.py develop`.

##Packages
Your version of this could look something like this one does:

~~~
|-- tmcmarkdown
|   |-- extensions
|   |   |-- gifv.py
|   |   `-- __init__.py
|   |-- tests
|   |   |-- __init__.py
|   |   `-- testgifv.py
|   `-- __init__.py
|-- LICENSE.md
|-- README.txt
`-- setup.py
~~~

This setup lets us `tmcmarkdown.extensions.gifv` and then use classes in there, ostensibly `GifV`.

The contents of `setup.py` requires

[On to testing.](lets-make-a-terrible-markdown-extension-pt2-getting-testing)
