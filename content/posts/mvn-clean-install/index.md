---
title: "Don't just blindly run mvn clean install..."
date: 2020-10-13T17:03:00+02:00
draft: false
tags: [mvn, java, build system, reactor]
categories: [programming]
resources:
    - name: featured-image
      src: craftsman.png
lightgallery: true
code:
    maxShownLines: 50
---

## TL;DR

Believe me you almost never want to run the `install` goal.

If you want to create artifacts use this command:

```bash
mvn package
```

throw in the occasional `-D skipTests` if you're feeling adventurous. :wink:

If you want to run tests (e.g. in a CI build) use this command:

```bash
mvn verify
```

If you're not convinced please keep reading.

## about that maven install goal

You see `mvn clean install` has been handed around for years as the goto command to "build the software". But actually it is probably doing more than you are looking for.

### we need to clean, or do we?

First things first: why do we prefix the command with `clean`? Well the target directory could hold outdated _stuff_ right? Can you actually remember a case where this was relevant? _Maven_ usually is smart enough to deal with whatever is left over in the target directory. Or should I say it is rather _"dumb"_ enough? _Maven_ tends to build more than is necessary because unlike _ye olde autotools_ it doesn't factor in file timestamps of source and target items to reduce the workload. One could argue that in the case of _java_ or _JVM_ based projects this would be a tough task anyway. Not every ".java" file maps clearly to a pre-known output file. Think about inner or anonymous classes.

What the `clean` goal is doing is to try to delete every target folder in every module of your project. Only for it to be created again immediately by the next lifecycle phase. So my advice would be __if you don't necessarily know why you would use clean, just don't__ it's gonna be fine in the most cases. 

### install always works, right?

If you've ever had the pleasure of working with so called _"multi module"_ projects you know the pain. _Maven_ really isn't all that intuitive when it comes to who inter-project dependencies are resolved. Sooner or later someone will tell you "well duh, you need to install the depencies to your local repo" and from that point of time on running the ocassional `mvn clean install` is just daily business.

Take a look at the following simplified project structure:

{{< image src="multi_module.png" caption="a maven multi module project" >}} 

We have a root project (usually called "parent") and 3 child projects "core", "cli" and "web". We can also see that both "web" and "cli" depend on the "core" module. That's a pretty common structure where most of the code probably resides in the "core" module while different aspects like a web frontend in the web module might be relying on the core functionality.

Now the tricky question: if you change something in the core module, will the dependant cli or web modules automatically pick up those changes on the next build? As everything in IT the answer to that question of course is: __it depends!__

The very unintuitive default for _maven_ is to look up dependencies in your local _maven_ repository. That is the `~/.m2/repostory` thingy. This default also applies to dependencies if they happen to be located in the very same multi module project. Now the quick fix of course is to run `install`. The problem with that of course is that now you have a fixed (snapshot?) version of your code built and copied to your local repo. This will not auto-update. Once you make further changes your dependant modules again will not _"see"_ the changes. This can lead to frustrating cycles of debugging, remembering that you forgot to do the _"install dance"_ and having to do the same thing all over again.

### fire up that Reactor

There is a way to tell _maven_ to actually look for dependencies in the local project first. _Maven_ does this by activating a thing called __the Reactor__. The funny thing is that whether _maven_ actually uses the _reactor_ or not is by default bound to the goal (or rather lifecycle phase) that you are trying to execute. Goals like `install`, `package` and `verify`use the _reactor_ by default. This is why you might have the impression that `install` is the solution to your dependency problems. Goals like `compile` on the other hand don't use the _reactor_ by default. They will gladly use outdated dependencies straight out of your local repo.

So what if we want to build maybe only the web artifact of that example project mentioned above? We will need to activate the reactor, to get the latest `core.jar` bundled into our `web.war`. There are two ways to do this without doing an `install` and therefore not littering our system with unnecessary artifacts:

 1. we could run a complete package from the parent (building more than necessary):
 ```bash
cd root
mvn package
 ```
 2. or we explicitly tell _maven_ to build the web project and its dependencies while activating the _reactor_
 ```bash
cd root
mvn compile -pl web -am
```

{{< admonition type=info title="reactor protip" >}}
The take away here is that simply specifying a "project list" via `-pl` is enough to use the reactor. 
{{< /admonition >}}

Ever noticed the nice build summary _maven_ outputs by default? It's actually called a _reactor_ summary as you can see below:

```bash
[INFO] Reactor Summary for parent 0.0.1-SNAPSHOT:
[INFO]
[INFO] parent ............................................. SUCCESS [  0.011 s]
[INFO] core ............................................... SUCCESS [  1.590 s]
[INFO] cli ................................................ SUCCESS [  1.518 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.273 s
[INFO] Finished at: 2020-10-13T18:09:21+02:00
[INFO] ------------------------------------------------------------------------
```

## Remarks

I once had a blog post with the exact same title on Marco Behler's [Blog](https://www.marcobehler.com/blog) back before the big redesign. By now Marco replaced that blog post with an extensive guide on Maven, you can find the link to that guide in the links section.

## Links 
 - In dept guide about "mvn clean install" by [Marco Behler](https://www.marcobehler.com/guides/mvn-clean-install-a-short-guide-to-maven)
 - Photo by [Milan Popovic](https://unsplash.com/@itsmiki5?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/craftsman?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)
 