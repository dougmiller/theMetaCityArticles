Title: Pushing to multiple git repositories
Tags: git
Blurb: Avoiding single point of failure when pushing git to a single repo (github) by sending to multiple remotes at once.

git is a distributed version control. Distributed. While you can have a single `master` remote it can be very handy to push changes to several remotes while onyl pulling from one. Here is how I do it.

## Open up the `.git/config` file in your repository

Inside this there are several sections that are of interest. The following is a bog standard file for a newly `initialise`'d repository.

```config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
```

Great stuff.

Now lets add a remote to push changes to:

`git remote add github git@github.com:dougmiller/theMetaCityArticles.git`

which now gives us

```config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "github"]
	url = git@github.com:dougmiller/theMetaCityArticles.git
	fetch = +refs/heads/*:refs/remotes/github/*
```

We can of course add as many of these as we like

`git add remote tmc doug@themetacity.com:theMetaCityArticles.git`

which unsurprisingly add a second remote

```config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "github"]
	url = git@github.com:dougmiller/theMetaCityArticles.git
	fetch = +refs/heads/*:refs/remotes/github/*
[remote "tmc"]
	url = doug@themetacity.com:theMetaCityArticles.git
	fetch = +refs/heads/*:refs/remotes/tmc/*
```

And see what git thinks about where it can push and pull to: `git remote -v`

```
github	git@github.com:dougmiller/theMetaCityArticles.git (fetch)
github	git@github.com:dougmiller/theMetaCityArticles.git (push)
tmc	doug@themetacity.com:theMetaCityArticles.git (fetch)
tmc	doug@themetacity.com:theMetaCityArticles.git (push)
```

## You have `commit`'d
After doing a commit, to get the changes to each remote you could do:

`git push github master && git push tmc master`

This will get the data to both remotes but is a bit clunky. A cleaner solution can be to edit a remote in the config to have two push locations.