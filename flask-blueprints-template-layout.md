Title: A helpful way to layout your flask template files for blueprints
URL: a-helpful-way-to-layout-your-flask-template-files-for-blueprints
Blurb: Flask blueprints are super helpful. The templates contained within them can collide with other ones. Setting them up this way is a good way to avoid that. 
Tags: Flask,Programming

If you setup the barest of minimum Flask projects you might have a folder layout like this (adapted from the official flask tutorial):

<folder layout>

This works great:

<code of the return render>

Which results in:

<it's result>

Brilliant.

Lets expand this example in include blueprints:

<folder layout>

Render out a template:

<return render template>

Which nets us this result:

<Reuslt>

Wait, what?

Why is the top level