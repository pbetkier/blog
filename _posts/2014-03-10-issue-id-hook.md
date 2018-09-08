---
layout: post
title: Hook for issue id in commit message
date: 2014-03-10
---

In my current project we have a policy that each commit should relate to an existing
JIRA issue. Also, we work with feature branches with user story ids present in
heir name. So it's reasonable to automatically prepend the issue id to the
commit message using a git hook.

I quickly searched for existing hooks, but they were either to complex, with more
advanced JIRA integration, or too simplistic – not handling the corner cases. I also
wanted to learn more about git hooks so I decided to
[write one myself](https://github.com/pbetkier/add-issue-id-hook).

It's pretty simple, let's say your project id in JIRA is EXAMPLE and you work on
a user story with id EXAMPLE-134:

```
$ git checkout -b EXAMPLE-134_some_story
```

Each commit you make will have the user story id prepended to the message:

```
$ git commit "Added some code."
$ git log
    ...
    EXAMPLE-134 Added some code.
    ...
```

By the way, I guess my hook is the only hook for prepending issue ids to commit
messages with a full test coverage ;)