Title: Not reading the fucking manual leads to pain. Again.
URL: not-reading-the-fucking-manual-leads-to-pain-again.
Blurb: Did not read the manual and spent a few weeks writing code to handle file parsing that is totally unnecessary. Because I am an idiot. On the plus side, I get to delete code.
Tags: Python,Markdown

The articles for this site are written in Markdown and then parsed and inserted by Python.

I was manually parsing then constructing the articles from an in-house format that was a bit flaky. The format looked something like this:

```markdown
# Title of the article

The blurb/summary shown on the index pages.

###################
Type: blog/workshop
Tags: Comma,delimited,list
Parent: id of series parent
###################

Article proper starts
```

Problems arose immediately: does the first title line have a space or not? Is there always a blank line following that? Having to manually isolate the meta fields within the ```####################``` fenced blocks is dumb. Is there another blank line after that? Having to remember that this is the format and parse the file several times to make sure I have done it correctly.

Nonsense. Just nonsense.

Imagine my surprise then, that when I was reading the documentation of the extensions' library that someone else had run into this issue, written it as an extension and published it as part of the main library. Who would have thought? Apparently not me.

Anyway, this transforms the code in two different ways: the files change to be in a much more palatable and stable format, and the files can now be read directly from disk and don't require manually reading them in (also in the docs).

Goes from this:

```python
""" Setup etc done previously above """

class Article:
    def __init__(self, file_object):
        self.file_object = file_object
        self.markdown_processor = Markdown()

        self.meta = {}

        if file_object.id:
            self.meta['id'] = int(file_object.id.strip('-'))

        split_article = self.file_object.raw_data.split('####################')

        self.head = self._extract_head(split_article[0])
        self.meta = {**self.meta, **self._extract_meta(split_article[1])}

        if self.meta.get('id'):
            self.article = models.Article.query.get(self.meta.get('id'))
            self.article.id = self.meta.get('id')
            print("Article exists. Updating...")
        else:
            self.article = models.Article()
            print("New article.")

        self.article.title = self.head['title']
        self.article.url = self.head['url']
        self.article.blurb = self.head['blurb']
        self.article.text = self.markdown_processor.process(split_article[2])

        self.article.parent_id = self.meta.get('parent')
        self.article.type = self.meta.get('type', 'blog')
        self.article.blurb = self.head.get('blurb')

        db.session.add(self.article)

        print("Removing tags")
        self.article.tags = []
        db.session.commit()

        for tag in self.meta['tags']:
            t = models.Tag.query.filter_by(tag=tag).first()

            if t is None:
                db.session.add(models.Tag(tag=tag))
                db.session.commit()
                t = models.Tag.query.filter_by(tag=tag).first()

            self.article.tags.append(t)
            print("Added tag: " + tag)

        db.session.commit()
        print("Article saved.")

    def _extract_head(self, raw):
        self.lines = raw.splitlines()

        # Strips of the '# ' (hash-space) at the start of the title
        self.title = self.lines[0].strip()[2:]

        print('Title: ' + self.title)
        if self.title is '':
            print("Title is empty. Do you have blank lines at top of file?")
            sys.exit(6)

        self.url = self.title
        self.url = re.sub(" ", "-", self.url)
        self.url = re.sub("\.", "-", self.url)
        self.url = re.sub(":", "-", self.url)
        self.url = re.sub("-+", "-", self.url)
        self.url = self.url.replace('-+', '-')
        self.url = self.url.replace('.+', '-')
        self.url = self.url.replace(':+', '')
        self.url = self.url.lower()

        self.blurb = self.markdown_processor.process("\n".join(x for x in self.lines[1:] if x))

        return {'title': self.title, 'url': self.url, 'blurb': self.blurb}

    def _extract_meta(self, raw_header):
        self.meta = {}

        self.lines = iter(raw_header.splitlines())

        for line in self.lines:
            s = line.split(':')

            if len(s) > 1:
                self.meta[s[0].lower()] = s[1]

        if self.meta['tags']:
            self.meta['tags'] = self.meta['tags'].split(',')
        else:
            self.meta['tags'] = []

        return self.meta
```

to this:

```python
class Article:
    def __init__(self, text, meta):

        if meta['id']:
            article = models.Article.query.get(meta['id'])
            print("Article exists. Updating...")
        else:
            article = models.Article()
            print("New article.")

        article.title = meta['title']
        article.url = meta['url']
        article.blurb = meta['blurb']
        article.parent_id = meta['parent']
        article.type = meta['type']
        article.text = text

        db.session.add(article)

        print("Removing tags")
        article.tags = []
        db.session.commit()

        for tag in meta['tags']:
            t = models.Tag.query.filter_by(tag=tag).first()

            if t is None:
                db.session.add(models.Tag(tag=tag))
                db.session.commit()
                t = models.Tag.query.filter_by(tag=tag).first()

            article.tags.append(t)
            print("Added tag: " + tag)

        db.session.commit()
        print("Article saved.")
```

There is a supporting file class that handles opening and extracting the content from the file. Previously it was just a file to get the raw string and pass of processing however this change now has that class do the extraction and mapping of meta (sanity chck for what is present and what is not) and then return the md object.

```python
class File:
    """
    File actions
    """

    def __init__(self, filename):
        self.text = None
        self.md = None

        try:
            with open(r'articles/' + filename) as file_contents:
                self.md = markdown.Markdown(
                    extensions=[GifV(), 'meta', 'fenced_code', 'codehilite', 'toc']
                )
                self.text = self.md.convert(file_contents.read())
        except FileNotFoundError:
            print("I was not able to find the file to open")
            exit(7)

        id_regex = re.search(r'^\d\d\d-', filename)

        if id_regex:
            self.md.Meta['id'] = id_regex.group(0).split('-')[0]
        else:
            self.md.Meta['id'] = None

        try:
            self.md.Meta['title'] = self.md.Meta['title'][0]
        except KeyError:
            print("Articles need a title")
            exit(8)

        try:
            self.md.Meta['url'] = self.make_url_from_title(self.md.Meta['title'])
        except KeyError:
            pass

        try:
            self.md.Meta['blurb'] = markdown.markdown(self.md.Meta['blurb'][0])
        except KeyError:
            print("Articles need a blurb")
            exit(8)

        try:
            self.md.Meta['parent'] = self.md.Meta['parent'][0]
        except KeyError:
            self.md.Meta['parent'] = None

        try:
            self.md.Meta['type'] = self.md.Meta['type'][0]
        except KeyError:
            self.md.Meta['type'] = 'blog'

        try:
            self.md.Meta['tags'] = self.md.Meta['tags'].split(',')
        except KeyError:
            self.md.Meta['tags'] = []

    @staticmethod
    def make_url_from_title(title):
        url = title
        url = re.sub(r" ", "-", url)
        url = re.sub(r"\.", "-", url)
        url = re.sub(r":", "-", url)
        url = re.sub(r"-+", "-", url)
        url = re.sub(r"-+$", "", url)
        url = url.lower()
        return url
```

In this section of that code:

``` python
with open(r'articles/' + filename) as file_contents:
    self.md = markdown.Markdown(
        extensions=[GifV(), 'meta', 'fenced_code', 'codehilite', 'toc']
    )
    self.text = self.md.convert(file_contents.read())
```

I need to open a file, read the content and now because there is no longer any custom parsing, convert straight away. md provides a method ```convertFile()``` that could take care of this step, but it can only outout to standard out or to another file. I'll see about time to submit a patch for that.