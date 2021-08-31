Title: Time to rebuild the MetaCity
URL: time-to-rebuild-the-metacity
Blurb: the MetaCity backend is getting rebuilt. Here are some of the details.
Type:blog
Tags:theMetaCity

## TLDR: too annoying to maintain; modern frameworks add value

The MetaCity was originally started in early 2004 as part of a uni assignment. Initial used for learning and playing around to learn Java and database nonsense it evolved over the years to become a fairly basic blogging platform.

It is built using Java with JavaBeans and the JSTL on top of PostgreSQL and Tomcat reverse-proxied behind apache2.

Over the last fifteen years or so of building upon and maintaining the system, it has become increasingly harder to maintain the environment around developing, testing, deploying and serving. This is partly due to expanded scope of other projects taking up time and just getting sick of the time devoted to operations.

To that end, the plan is to rebuild the MetaCity in Python with Flask to bring it into line with the MetaCity Media. This unifies development to one language and one platform, reducing cognitive overhead.

An added benefit is that this will finally bring the MetaCity into the world of actual frameworks with all the benefits they bring. Notably, this includes an ORM for the first time, up to date language version, and an easier to access and manage third-party package management (pip).

I'm also going to rework how the workshop is build and managed so that it links in a bit nicer and makes showing it in other sections much easier too.

Processing articles remains the same but inserting and updating might (or might not, dunno yet) change as integrating would in theory be easier.

## Why?
### Play
This is the main reason. It is an opportunity to just see what I learn and experience with a different setup and development process. Python is interesting to work with, and I hope to have fun while also building something useful doing this.

### Server maintenance
Reverse proxing behind tomcat for one project has become too annoying. This adds another service that needs to be updated, configured and monitored. It also means that Java needs to be updated and maintained.

### The MetaCity is custom code all they way down
That was nice at the time, but the warts are showing. Here is what a rebuild hopes to solve:

#### No ORM
This one is simply a time saver. Currently, the ORM is a significant portion of the LOC and this boilerplate is a pain to maintain. While straightforward to do the mappings, it is the migrations and updates to the objects in templates that get dumb.

#### Template abstraction is messy
Files are manually included and messily combined to make the pages show correctly. The includes and override mechanic of Flask is much nicer to use.

#### Plugin/third party code is easier
The MetaCity's plugins are all manually updated (via searching) and inserted into the appropriate directory manually. Flask uses pip and is much easier to maintain. Have not run into an issue with a feature being supported in Java and not in Python but the MetaCity is not doing anything too crazy.

## What would be changed/new?
Not too much really. This is basically a one for one feature wise rebuild. The main difference is (planned at least) to combine the tagging system to integrate the blog and workshop together, making indexes and notification of updates easier (as well as the document import). There are some schema changes to make this happen but for the most part there should be no data loss and only transparent changes to the front end.

## Other
This presents an interesting opportunity to do a comparison of speeds etc which might be interesting.

It will also be nice to time-lapse the whole thing to see what kind of dev time this takes.
