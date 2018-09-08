---
layout: post
title: Keep your .gitignore clean as well!
date: 2015-02-03
---

We’re all crazy about keeping our git history clean. Arguments over the styling
of commit messages, choosing merge vs rebase, even leaving trailing commas in
some languages for better diffs – these are the things we are accustomed to.
Yet, for some reason, we don’t pay the same attention to our `.gitignore` files.

## Messy gitignores
It’s too common to see `.gitignore` files that are a complete mess. Basically,
they are just a compilation of all the files that any developer, at any point
in time in the project, didn’t want to track. It’s especially visible when
developers work in different environments: different operating systems, IDEs
and other tools. We’ve all seen things like this:

```
build/
.gradle/
*.iml
*.ipr
*.iws
.idea/
some_weird_file_nobody_remembers_why_is_here.xml
.settings/
.project
.classpath
some/local/directory/one/developer/didnt/want/to/commit/
nb-configuration.xml
nbactions.xml
some_garbage.log
.nb-gradle/
rebel.xml
*.swp
.DS_Store
even_more_garbage.log
...
```

Looks familiar? This list is basically append-only, since you don’t know
if the file you consider deleting isn’t important to somebody. Also, if you
create new repositories regularly (doing microservices, right?) there is a big
temptation to just copy & paste most of this garbage to the new project
(please don’t). Sure, maintaining such `.gitignore` files is not the end of the
world, but we can do better.

## Remedy – global gitignores
The remedy is to start using global `.gitignore` files. Each developer should
define his/her own global `.gitignore`, e.g. in his/her home directory, that
affects all the repositories on his/her machine. Here's how to add a reference
to this global `.gitignore` in a git config:

```
$ git config --global core.excludesfile ~/.gitignore_global
```

Each developer puts all the ignore rules specific to his/her environment,
e.g. his IDE or operating system, in `.gitignore_global` file. As a result
we can separate project-specific ignores – these are tracked in the project
repository, from environment-specific ones – each developer handles them by
himself.

Now the project’s `.gitignore` file may look more or less like this:

```
build/
.gradle/
```

and e.g. my own global `.gitignore` like this, since I use IntelliJ and vim
on OS X:

```
.idea/
*.iml
*.swp
.DS_Store
...
```

You can also define your local ignores specific to a single project, just
put them into `.git/info/exclude` in the repository. They won’t be tracked.

## Summary

Polluted `.gitignore` files are not a major issue in software development,
but still happen well too often just to be ignored (no pun intended). All
we have to do is to separate ignores specific to the project from ignores
specific to developer’s environment. By the way, you may find
[this service](https://www.gitignore.io/)
very useful when creating your own global ignores – it generates ignore rules
from the list of tools that you use. Happy ignoring!
