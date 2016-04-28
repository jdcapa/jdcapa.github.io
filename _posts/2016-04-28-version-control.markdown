---
layout: post
title:  "git and SVN repository management"
date:   2016-04-28 19:11:19 +0200
categories: git svn version-control python coding
---

Motivation
----------

I recently started to use
[git](https://en.wikipedia.org/wiki/Git_%28software%29)
 and to some extent
 [SVN](https://en.wikipedia.org/wiki/Apache_Subversion)
 in order to subject my
 code development to the many
 [advantages of version control](https://stackoverflow.com/questions/1408450/why-should-i-use-version-control).
Now, most of you who develop code might have come in contact with this (if not
 I highly suggest you do; the free
 [Pro Git book](https://git-scm.com/book/en/v2) is an excellent start).
The way it works is that you usually have a *coding project* in a folder
 which will, if you initialised it with some version-control contain, next to
 the actual code, a *dot-folder* (e.g. `.git` or `.svn`).
Often, depending on the circumstances, the *project folder* is tucked away
 neatly (or chaotically) somewhere in some deeper folder structure.
In most cases this is great but sometimes this can cause issues and for the
 following reasons I like to keep my dot-folders in a separate place on my hard
 drive.

    + If the coding folder itself is synced (for backup purposes) and the
    project is large:
        - separating the dot-folder to be extra precocious
        - the dot-folder can be large and unnecessary syncing can take some time
    + I would like an overview of all my projects

Implementation
--------------

As I said, the dot-folders need to be stored in **one** place (Obviously, a
 symbolic dot-folder link to this location is created in their stead).
Personally, I found `$HOME/.VersionControl` to be most suitable.
Furthermore, there should be a `$HOME/.VersionControl/projects.yaml` file
 which contains the details to all my version-controlled projects.
This is realised through a little python script I wrote and it's called
 [vcm (VersionControlManagement)](https://github.com/jdcapa/VersionControlManagement).

If you jump into a project folder called `ProjectA` controlled for example by
 git and you type

{% highlight bash %}
vcm -vc git -m
{% endhighlight %}

you'll find your project to be added to the `projects.yaml` file and your
 `.git` folder moved to `$HOME/.VersionControl/ProjectA-git` and a symlink
  pointing to it: `.git -> $HOME/.VersionControl/ProjectA-git`.

Adding more features is easy, e.g.:

{% highlight bash %}
vcm -g -u jdcapa -vc git
{% endhighlight %}

where you now also told the `projects.yaml` file that this project is a github
 project to be found under the username jdcapa.

This might be interesting for future automation. For now I'm happy with the
 separation and overview feature.

*-jdcapa*

