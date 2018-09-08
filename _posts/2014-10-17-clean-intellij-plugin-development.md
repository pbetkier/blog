---
layout: post
title: Clean IntelliJ plugin development
date: 2014-10-17
---

> This post is out of date since JetBrains has greatly improved the plugin
> development experience. Visit
> [IntelliJ Platform SDK guide](http://www.jetbrains.org/intellij/sdk/docs/welcome.html)
> for more information.

IntelliJ IDEA and its cousins from JetBrains are probably the best IDEs currently
available on the market. One of the many reasons for that is the large number of
high quality plugins to choose from. And it's not that hard to build one yourself!

It's pretty straightforward to start with writing a plugin – there is a short
[getting started guide](http://confluence.jetbrains.com/display/IDEADEV/Getting+Started+with+Plugin+Development)
available on JetBrains confluence, plus some poorly structured materials
from JetBrains [here](http://confluence.jetbrains.com/display/IDEADEV/PluginDevelopment)
and [here](http://confluence.jetbrains.com/display/IDEADEV/Plugin+Development+FAQ).
Sadly, apart from that, you're on your own.

Setting up your project for plugin development up to the standard you usually have is
not that easy though. In this blogpost I'll show you my ideas on how to do it the
clean way. If you'd also feel ashamed publishing your plugin on github without
e.g. build automation, read on.

## Make it clean

JetBrains doesn't help you with setting up your project environment. That's why
dependency jars are committed into the repository in most of the plugin projects
you can find on github. Most of the time, there is also no CI configured to run
the tests. That's sad and we can do better.

### Resolve dependencies with Gradle

First of all, I'd like the dependencies to be resolved automatically like they
should. No more jars committed to the repository. Normally you would just
import your Gradle project into IntelliJ and that's it. You cannot do it
in this case though. The problem is that a plugin project has a special `plugin`
type in IntelliJ and importing the project as a Gradle project doesn't set it.

Luckily there is another option for Gradle-IntelliJ integration, that is to
generate idea files yourself using an `idea` Gradle plugin. With this snippet in
your `build.gradle`, running `gradle setup` will generate proper idea project files.
All you have to do now is to open the project in IntelliJ.

```
apply plugin: 'idea'

...

idea.project.ipr {
    beforeMerged { project ->
        project.modulePaths.clear()
    }
}

idea.module.iml {
    withXml {
        it.node.@type = "PLUGIN_MODULE"
    }
}

task setup {
    dependsOn ideaModule, ideaProject
    doLast {
        copy {
            from '.'
            into '.idea/'
            include '*.ipr'
            rename { "modules.xml" }
        }
        project.delete "${project.name}.ipr"
    }
}
```

After opening the project for the first time, IntelliJ will ask you should Gradle
integration be added, just click Yes. You will also have to link your local IntelliJ SDK.
[This blogpost](http://arhipov.blogspot.com/2013/10/managing-dependencies-for-intellij-idea.html)
helped me achieve all that.

### Run your tests from the command-line

Being able to run your test from the command-line is crucial for setting up a CI.
The problem is the dependency on IntelliJ SDK, more precisely on some of the IntelliJ jars.

Since you configured your IntelliJ SDK, you will have all the jars on the classpath
when compiling from IDE. They won't be available when compiling from the command-line
though. You need to attach them in from a different configuration, a little trick that
prevents IDEA Gradle plugin from adding (and duplicating) them to the classpath in the
IDE, but making them available when compiling from the command-line.

```
configurations {
    provided
}

dependencies {
    ...
    provided fileTree(dir: 'idea-libs', include: '*.jar')
}

sourceSets {
    main {
        compileClasspath += configurations.provided
    }
    test {
        compileClasspath += configurations.provided
        runtimeClasspath += configurations.provided
    }
}
```

What is `idea-libs`? It's a symlink in the project root that links to IntelliJ's
`lib/` subdirectory. Don't track this file in your SCM since every developer has a different path.

### Setting up travis

Now that we can run the tests from the command-line we're ready to set up our CI. We need to
provide `idea-libs` symlink on the CI agent for the compilation to succeed. My brutal
solution is to simply download IDEA and create the symlink before building the project on CI.

```
#!/usr/bin/env sh

IDEA_VERSION="13.1.2"

wget -O idea.tar.gz http://download-cf.jetbrains.com/idea/ideaIC-${IDEA_VERSION}.tar.gz

tar xf idea.tar.gz
ln -sT `find . -name 'idea-IC*'`/lib idea-libs
```

Having completed all these three steps we can now happily include the "build passing"
label in our readme!

## Make it easier

So far we discussed how to make our development clean, with running tests on the
CI and all that. Now let's look at ways to make it easier.

### Fast prototyping

As I mentioned, the documentation for plugin development is scarce, so what you find
yourself doing almost constantly is what I call *adventure coding*. You try to recall
a feature or a plugin in IntelliJ that does that something you look for, search for
its code, dive into it and borrow the ideas.

That's why you often want to check how some piece of code would behave. But running your
plugin takes quite some time. To shorten the feedback cycle, you can use
[liveplugin](https://github.com/dkandalov/live-plugin), an awesome IntelliJ plugin
for writing plugins at runtime. You can test some snippets in there and use them in
your plugin code once you are satisfied with the results.

### Use Groovy

Groovy is an excellent language for the JVM, especially for small projects like an
IntelliJ plugin. Actually, IntelliJ 13 already contains Groovy 2.2 jar, so you can
write your plugin in Groovy without adding it as an external dependency and increasing
the plugin final file size. Throw in [spock](http://spockframework.org/) for
unit tests and you're all set.

## Wrapping up

Writing IntelliJ plugins is fun, but not straightforward to do right. There are no
well-known good practices to follow, rather everyone comes up with his own ideas.
I presented the ideas I came up with when working on
[IntelliJ Synonyms](https://github.com/pbetkier/intellij-synonyms) plugin, but they
surely could be improved – please contact me if you know how!
