Title: Intermediate Flask
Blurb: So you can spin up a quick Flask app and run it in dev mode. What do now?

These are some things to do to get you moving into the intermediate Flask realm.

## Templates in redundant folders

Flask will use the first template name that it matches with. With small apps that have few templates, the template used is unlikely to have a name collision with anything else, especially once you start getting folders involved.

<pre>
<code>
templates/
├─ index.jinja2

// Used like so: render_template('/index.jinja2')
// All is working well
</code>
</pre>

With a sub-folder:

<pre>
<code>
templates/
├─ index.jinja2
├─ items/
│  ├─ index.jinja2

// Used like so: render_template('/items/index.jinja2')
// All is working well still; no name collision
</code>
</pre>

But once we start getting more advanced with our setup we might run into a situation where (because Blueprints) there might be a collision:

<pre>
<code>
blueprint1/
├─ templates/
│  ├─ index.jinja2
blueprint2/
├─ templates/
│  ├─ index.jinja2

// If we: render_template('/index.jinja2')
// Which file are we using?
</code>
</pre>

The solution to the above is to add the blueprint name as a sub-folder in the template structure:

<pre>
<code>
blueprint1/
├─ templates/
│  ├─ blueprint1/
│  │  ├─ base.jinja2
blueprint2/
├─ templates/
│  ├─ blueprint2/
│  │  ├─ base.jinja2

// If we: render_template('/blueprint1/index.jinja2')
// The ambiguity has been removed
</code>
</pre>

If we accept that there is slightly more verbosity in the `render_template` function, we gain a few wins along the way. Firstly, we now know exactly the template we want to render. "But," I hear  you say, "I mostly often only want to render the templates in my blueprint and this adds an annoying redundancy and makes the function call look weird. Can't you just make it so that it defaults to this blueprint and I supply another parameter to the function if I want a different blueprint's templates?" To which I respond, "No, too bad. Flask doesn't work that way. Not sorry."

This features can be a bit annoying like that but it does become really helpful once you want to call a template in a different blueprint: just call with a different prefix template name.

<pre>
<code>
render_template('/blueprint1/index.jinja2')
render_template('/blueprint2/index.jinja2')
// Whatever you need, go nuts
</code>
</pre>

//todo confirm how this works

## Model inheritance (SQL Alchemy)
What if your 

