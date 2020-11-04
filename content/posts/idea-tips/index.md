---
title: "5 + 1 tips for new IntelliJ IDEA users"
date: 2020-11-04T19:34:14+02:00
draft: false
tags: [java, intellij, idea, ide]
categories: [programming]
resources:
    - name: featured-image
      src: logo-intellij-idea_centered.png
lightgallery: true
code:
    maxShownLines: 50
---

## Why IDEA?

It seems like a lot of people have migrated to [IntelliJ IDEA](https://www.jetbrains.com/idea/) over the years. As a long time _IDEA_ user I can attest there are many reasons for that. So if you're willing to make the jump here are some tips to easy your life.

Of course your mileage may vary, and you don't necessarily need to agree with my opinions. That's completely fine. :wink:

## Import Maven Projects like a Pro

I don't really know why that is the case but over the years I've found that there is one surefire way to correctly import maven projects, and it is not obvious. To import a maven project that you already have sitting ready on your local drive (e.g. just checked out from _SCM_) follow these steps:
  1. click "File" / "Open..."
  2. navigate to the project folder
  3. select the :(far fa-file-alt):`pom.xml` file
  4. choose "Open as project"
  5. give IDEA some time to create the project, run its indexer etc.

{{< image src="pom_xml.PNG" caption="Open the pom.xml" >}}

I've found this seems to set up a maven project correctly from the get go. All other methods of importing project like "import from existing sources" or just simply opening the project folder may end up leaving you with a non-maven project that you have to adjust manually to start the _Maven_ integration.

## Don't ignore the Event Log

Every time something significant happens, like opening a new project, a new plugin update is available, etc. _IDEA_ will inform the user with a small bubble like info on the lower right corner of the screen. I've seen people handle those as some kinds of _"bad news"_ error message or even like spam. A lot of people seem to like to ignore whatever their IDE is trying to tell them. The problem is if you want to get the most out of your IDE you will need to pay attention to those messages.

One repeating example I see is when _IDEA_ detects spring configurations. The IDE is super smart with code completion and refactorings if you let it know about your spring context info. It will even autodetect most if it but you as a user need to take the minute to think about whether this context info is correct or not. It usually just requires you to click on a link. Still a lot of people seem to be annoyed by the popup, don't read it at all, never click the link, and therefore miss out on all the spring integration features.

 So my tip is: look at what _IDEA_ is trying to tell you. It will usually improve your life. Should you miss a balloon notification though, don't worry. They are all there in what is called the _"Event Log"_ below down right. You can still click on the links there.
 
 {{< image src="event_log.PNG" caption="Event Log" >}}

## Forget about Workspaces

If you've used other IDEs before you have grown accustomed to the notion of a _"workspace"_ folder. Usually you can add a lot of different projects to your IDE workspace and then just selectively open and close those depending on your requirements. The rough translation of this concept into _IntelliJ IDEA_ would be like this:

 | common term | corresponding IDEA concept |
 | ----------- | ---------------------------|
 | workspace   | project                    |
 | project     | module                     | 
 
So one might be tempted to keep using the tried old workspace scheme, now just mapping it to _IDEA_ projects. While this might work for simple cases it tends to get confusing fast. _IDEA_ does not have a notion of opening or closing individual project modules. Projects really act as containers for modules that belong together in classical sense. The solution is easy: just use different projects and open those as separate windows if necessary.

## Make use of the SCM Integrations

Again as if with the preceding point if you're used to other IDEs you will usually have a experience with flaky strange SCM integrations that in the best case are only a wrapper around command line tools, and in the worst case add their own idea of how to do SCM on top. The seasoned developer sooner or later learns that proper SCM invocations happen on the command line.

With _"IntelliJ IDEA"_ you get super smart and easy SCM integrations that enable you to do complex stuff like _git cherry-pick_ with a single button click while still enabling you understand what exactly is going on. Even if you don't have a use for those edge cases just remember these shortcuts:

 - `CTRL + t`: get scm changes (e.g. svn up, git fetch && git update, hg pull)
 - `CTRL + k`: open commit dialog
 
After you mastered those two shortcuts take a look down right, you should see the current branch you're working on. Click on that to open the super powerfull _"SCM operations"_ popup which enables you to quickly switch branches, check out tags etc.

 {{< image src="branch.PNG" caption="Branch Info" >}}

Can't find the SCM info in the status bar? Check your project settings and enable SCM integration. Maybe you missed a balloon popup informing you about the detected SCM and prompting you to enable the integration earlier.
 
The most powerful thing about the SCM integrations though is the interactive commit log view pictured below. Do people really want to call `git log` et al. on the command line all the time? There is literally no info the view in _IDEA_ wouldn't show you already.  
 
 {{< image src="git_log.PNG" caption="Git Log" >}}
 
## Immediately use those Keyboard Shortcuts

I know it is hard to re-train our minds to do things differntly but the key to fun in _IDEA_ is to use keyboard shortcuts. You should use the following ones immediately from the start:

| Keys | Action |
| ---- | ------ |
| CTRL + W | smartly increase text selection (just try it out) |
| F2 | jump between problems in the current file |
| ALT + ENTER | fix problem at cursor (may open context menu to chose the fix to apply) |
| CTRL + SPACE | force invoke _intelli sense_ |
| CTRL + SHIFT + SPACE | invoke __smart__ _intelli sense_ |
| CTRL + ALT + SPACE | complete current word context sensitively (if it is a class name add the fully qualified package etc.) |
| CTRL + V / CTRL + M / ...| extract variable / method etc. (or use postfix completion like .var instead) |
| CTRL + Q | show documentation for type at cursor |
| CTRL + H | show type hierarchy for type at cursor |
| SHIFT + F6 | rename _thing_ at cursor |
| STRG + F6 | refactor _thing_ at cursor |
| STRG + F1 | tell me more about the current problem |
| STRG + N | open class |
| STRG + SHIFT + N | open file |
 
## Keep Learning

I've been working with _IDEA_ pretty much every other day for more than a decade now. One would expect me to have mastered it by now. That couldn't be further from the truth so. One thing is that the people at [JetBrains](https://www.jetbrains.com) are still coming up with new ideas of how to improve their IDE, so the development never stops. The other thing is that a lot of really cool and pro functionality is usually not easy to find. Just today I discovered that the built in markdown support can even use _PlantUML_ diagrams, if setup correctly...

So definitely keep the "Tip of the day" enabled. You will learn new things every day. And on top of that think about maybe enabling plugins like the "Key Promoter", this will teach you what keyboard shortcuts you could have used to achieve whatever you just did by using the mouse. 

## Copyright
 - "JetBrains", "IntelliJ IDEA" and the IntelliJ IDEA Logo are trademarks and property of [JetBrains](https://www.jetbrains.com).
