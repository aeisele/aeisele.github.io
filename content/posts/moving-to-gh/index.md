---
title: "Moving to GitHub Pages"
date: 2020-08-05T23:00:12+02:00
draft: false
tags: [blog, hugo, static content, hosting, github]
categories: [personal]
resources:
    - name: featured-image
      src: Octocat-centered-streched.png
images: 
    - Octocat-centered-streched.png
lightgallery: true
diagram: true
---

## Hosting Hugo on GitHub Pages

As mentioned in my last [blog post](../bloggin-with-hugo) my goal was to move this entire homepage to be hosted on [github pages](https://pages.github.com/).

### TL;DR: My ToDo

 - [x] create blog locally
 - [x] add content
 - [x] setup github pages
 - [x] add github action to auto deploy
 - [ ] update DNS zone

### Hugo Build

The straight forward thing to turn this collection of _Markdown_ and other _resources_ into a proper webpage is to simply
have _Hugo_ built it. The command to achieve this is: :(fas fa-terminal): `hugo --minify`. This will spit out a directory ready to be served
by your trusty old :(fas fa-server): _httpd_ of your choice.

### Setting up GitHub Pages

So the next thing was to identify how to tell :(fab fa-github): _GitHub_ that I some static stuff hosted and no, it's not _Jekyll_. It turns
out there are three ways to _(trust me I've used them all)_:

| Repository  Name | What it does |
| --------- | ------------ |
| random default repo name | you can set up github pages but the base url of your page will include the repo name in the path: `https:://username.github.io/repoName/` |
| username | this is a special repository because it can store your personal `README.MD` that can be used to customize your github profile page. Other than that it follows the same rules as the option above. |
| username.github.io | Yes, the repo must have exactly this format if you want to host in what could be called the _root context_: `https://username.github.io/` | 

Of course people with better reading skills would have figured that out from the _GitHub Pages_ [documentation](https://pages.github.com/) already.

That's not all there is to do here. One actually needs to tell _GitHub_ which :(fas fa-code-branch): branch to use. This
is easy to locate in the actual repository settings.

{{< image src="gh-pages.PNG" caption="GitHub Pages repository config" >}}

Now we only need to create that branch and push it to _GitHub_ and the page should appear. Quick question for _Git_ pros, 
how do you creata a new empty branch that doesn't have any ancestors when you already populated your `main` branch (for
example if somebody comitted the source files for this blog already)? Turns out that thing is called an _orphan branch_.
This is how it can be done locally for a new branch called `gh-pages`: 

```shell script
git checkout --orphan gh-pages
git reset --hard
git commit --allow-empty -m "Initializing gh-pages branch"
git push upstream gh-pages
git checkout main
```
{{< admonition type=warning title="Warning" >}}
If you have git submodules in your working dir the `reset --hard` might get rid of the checked out files. You may have
to sync your submodules again after switching back to `main`.
{{< /admonition >}}

This worked liked a charm initally but one problem I faced was with the _Hugo_ `baseUrl` configuration. Locally in _development_
mode everything lives on the _root context_ so naturally this is how I crafted my urls in configs and css. But this is
not portable to a deployment under a subcontext (first repository naming option mentioned above). It seems like _Hugo_
is smart enough to detect that the _base url_ changed to something containing a path but this smartness doesn't apply
automatically in all instances. I also suspect that the [theme](https://hugoloveit.com) I'm using may have problems with
this kind of setup. So in the end only the last option (hosting on the root context) does really work sufficiently.

### Deploy on Push with GitHub Actions

[GitHub Actions](https://github.com/features/actions) is a really neat _newish_ feature of _GitHub_. It comes with a
marketplace and it didn't take me long to identify working [recipe](https://github.com/marketplace/actions/github-pages-action)
to apply to my repository. So one basically creates a _YAML_ :(far fa-file-alt): file through the provided _UI_ and
that's all that is needed to run actions on pushes. _Actions_ even provides the necessary security tokens etc. behind the
covers to enable creating new _Git_ commits automatically.

Mine at the time of writing looks like this:
```yaml
name: github pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.74.3'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

So this basically checks out each new push to the `main` branch, takes care that _Hugo Extended_ is available and runs
it to build the static page in the local `./public` :(far fa-folder): directory. This is then taken and put into a new
commit on the `gh-pages` branch which will in turn lead to an update of the static hosted page.

_Et Voila!_

### Coming soon...

_... to a nameserver near you_

Yeah so now the next thing to figure out and switch over is my old trusty _html_ based and _Apache_ hosted homepage.
This basically means setting up _A_ and _CNAME_ records to point to _GitHub_ and also configure the _GitHub_ side such
that the _vhost_ / _CNAME_ is recognized.
 
Hopefully this will turn this 

{{< image src="aedotcom.PNG" caption="the old andreaseisele.com" >}}

into that

{{< image src="aedotcomblog.PNG" caption="the brand new Æ blog" >}}

very soon.