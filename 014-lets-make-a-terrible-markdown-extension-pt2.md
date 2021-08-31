Title: Let's make a terrible Markdown extension pt2 - Getting testing
URL: lets-make-a-terrible-markdown-extension-pt2-getting-testing
Blurb: Second part of making a Python Markdown extension
Tags:Markdown,Python
Parent:12

In this page:

[TOC]

##Lets test
Testing can be helpful. So let's write some up.

##Assumptions
While we haven't written any working code yet, we can set up working tests so that we can measure out progress as we fill out the class.

The package is on the path somewhere (by running `python setup.py develop).

##GifV.py

Open up `GifV.py` and put in enough to not have the tests return an error.

~~~{.python}
class GifV(Extension):
    pass
~~~

##TestGifV.py

The test file itself is straightforward: call markdown with the extension registered and double-check the output is as expected.

No setup or teardown needed.

Loading the extension is straightforward with `markdown.markdown(provided, extensions=[list of extensions])`.

~~~{.python}
import markdown
import unittest
from tmcmarkdown.extensions.gifv import GifV


class TestGifV(unittest.TestCase):
    def testNotEnoughOptions(self):
        provided = '<gifv filename />'
        expected = '<p><gifv filename /></p>'
        self.assertEqual(expected, markdown.markdown(provided, extensions=[GifV()]))

    def testBasicOptions(self):
        provided = '<gifv sampleFileName extension />'
        expected = '<video autoplay="true" class="gifv" controls="false" loop="true"><source src="//assets.themetacity.com/gifv/sampleFileName.extension" /></video>'
        self.assertEqual(expected, markdown.markdown(provided, extensions=[GifV()]))

    def testBasicOptionsNoSpaceAtEnd(self):
        provided = '<gifv sampleFileName extension/>'
        expected = '<video autoplay="true" class="gifv" controls="false" loop="true"><source src="//assets.themetacity.com/gifv/sampleFileName.extension" /></video>'
        self.assertEqual(expected, markdown.markdown(provided, extensions=[GifV()]))

    def testBasicOptionsWithStart(self):
        provided = 'START <gifv sampleFileName extension />'
        expected = '<p>START <gifv sampleFileName extension /></p>'
        self.assertEqual(expected, markdown.markdown(provided, extensions=[GifV()]))

    def testBasicOptionsWithEnd(self):
        provided = '<gifv sampleFileName extension /> END'
        expected = '<p><gifv sampleFileName extension /> END</p>'
        self.assertEqual(expected, markdown.markdown(provided, extensions=[GifV()]))

    def testMultipleExtensions(self):
        provided = '<gifv sampleFileName extension,otherextension,thirdextension />'
        expected = '<video autoplay="true" class="gifv" controls="false" loop="true"><source src="//assets.themetacity.com/gifv/sampleFileName.extension" /><source src="//assets.themetacity.com/gifv/sampleFileName.otherextension" /><source src="//assets.themetacity.com/gifv/sampleFileName.thirdextension" /></video>'
        self.assertEqual(expected, markdown.markdown(provided, extensions=[GifV()]))

    def testTooManyOptions(self):
        provided = '<gifv sampleFileName extension extraUnneeded />'
        expected = '<p><gifv sampleFileName extension extraUnneeded /></p>'
        self.assertEqual(expected, markdown.markdown(provided, extensions=[GifV()]))

if __name__ == '__main__':
    unittest.main()
~~~

Unfortunately whitespace matters in the output here so the long strings have to remain.

Running the tests is straightforward, from the base of the module:

~~~
python -m unittest discover tmcmarkdown
~~~

will give you

~~~
.......
----------------------------------------------------------------------
Ran 7 tests in 0.021s

OK
~~~

Hooray.

Next up is [writing the bulk of the plugin](lets-make-a-terrible-markdown-extension-pt3-getting-coding).
