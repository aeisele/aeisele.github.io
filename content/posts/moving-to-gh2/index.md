---
title: "Moving to GitHub Pages, Pt.2"
date: 2020-08-06T15:49:20+02:00
draft: false
tags: [dns, static content, hosting, github]
categories: [personal]
resources:
    - name: featured-image
      src: massimo-botturi-zFYUsLk_50Y-unsplash-cut.jpg
images: 
    - massimo-botturi-zFYUsLk_50Y-unsplash-cut.jpg
lightgallery: true
---

## Further on Hosting Stuff with GitHub Pages

Let's repeat my ToDo list of the last [blog post](../moving-to-gh) here and see where we are:

 - [x] create blog locally
 - [x] add content
 - [x] setup github pages
 - [x] add github action to auto deploy
 - [x] update DNS zone
 - [x] enable comments
 - [x] enable share links
 - [ ] __add meaningful content__

Not too bad? But how did we arrive here?

### Hosting under custom domain

In theory that's quite easy. Once you have _GitHub Pages_ set up your page will already be available as `username.github.io`.
Nothing wrong with that, but I wanted it to act as my main homepage at the same time. This means convincing _GitHub_ to
recognize and accept requests for this host (`andreaseisele.com`) and to provide the content in the appropriate repo.

Two things need to be in place for this to work. A static file called :(far fa-file-alt): `CNAME` must be created in the
git repository. This file needs to hold the custom domain (think `www.example.com`). There is a settings UI on _GitHub_
itself to create this but in my case I had to let _Hugo_ supply the file in the deployment. Therefore it needs to be
created in the static folder.

Next is updating the _DNS Zone_ of the domain in question. This of course varies depending on the _DNS Provider_ but
what needs to be done is this:

 - have the so called _apex_ / root domain (`example.com`) point to GitHub's IP addresses
 - add a CNAME record for the www subdomain (`www.example.com`) pointing to `username.github.io`
 
There is a wonderful [answer](https://stackoverflow.com/a/9123911) on :(fab fa-stack-overflow): Stackoverflow going into all the necessary details and background info.

Your zone should look something like this in the end:

| Name | Type | Target |
| ---- | ---- | ------ |
| example.com | A | 185.199.108.153 |
| example.com | A | 185.199.109.153 |
| example.com | A | 185.199.110.153 |
| example.com | A | 185.199.111.153 |
| www\.example.com | CNAME | _username_.github.io |

{{< admonition type=warning title="Warning" >}}
The IP addresses mentioned above might change over time. Please refer to _GitHub_'s documentation for updates.
{{< /admonition >}}

### Enabling HTTPS

It turns out this step is easy. If everything is set up correctly, especially the _DNS_ stuff, all one needs to do is
to enable a setting in the repository config. Notice how it picked up the custom domain setting right out of the static _CNAME_ file.

{{< image src="gh-pages-https.PNG" caption="GitHub Pages repository config for HTTPS" >}}

This in turn will use [Let's Encrypt](https://letsencrypt.org/) behind the covers to generate a new certificate for the
custom domain. There is even a progress bar on this page initially for one to check the certificate status.

I went through some hiccups to get this to work for my site because of a combination of _DNS_ TTL / cache inconveniences
and me missconfiguring my initial _CNAME_ record. 

### Enabling Comments

Luckily the [theme](https://hugoloveit.com/) I'm using has support for various comment engines. Initially I'd decided to
not enable comments at all because I didn't want to store user input on my side and / or propagate it to third parties.
So services like [Disqus](https://disqus.com/) or [Commento](https://www.commento.io/) are out of the picture.
 
Still while browsing the configuration options available I came upon something interesting named [Utterances](https://utteranc.es/).
It works as a combination of something called a _GitHub App_, a bot service and some _JavaScript_ to be used on the target
page. The actual comments are stored as _GitHub Issues_ with a custom tag on a _GitHub_ repository one has to provide.

I've enabled _utterances_ in my global site configuration like this:

```toml
[params.page.comment]
    enable = true

        [params.page.comment.utterances]
            enable = true
            repo = "aeisele/andreaseisele.com-comments"
            issueTerm = "pathname"
            label = "blog-comment"
            lightTheme = "github-light"
            darkTheme = "github-dark"
```

If everything works according to plan you should be able to see a comments form right under this post. :crossed_fingers:

The problem is this will apply to all content. I'm really not keen on comments on my legal disclaimer pages, so one needs
to disable the global config again on static pages via the frontmatter like this:

```yaml
comment:
    enable: false
    utterances:
        enable: false
```

{{< admonition type=tip title="Tip" >}}
The _LoveIt_ theme doesn't enable dynamic content, like comments, when running in _development_ mode. To check whether the
comment section is rendered or not one has to run `hugo --minify` locally and examin the generated html in question manually.
{{< /admonition >}}

### Enabling (Social) Sharing

Again in my [first post](../blogging-with-hugo) I said I don't really understand the necessity of social share buttons.
Then again I figured it doesn't hurt to provide it nevertheless. This was really easy to do via configuration:

```toml
[params.page.share]
    enable = true
    Twitter = true
    LinkedIn = true
    HackerNews = true
    Reddit = true
    Xing = true
``` 

Technically this theme supports a lot more sharing options but I limited it to sites I use myself. You will not find
a _Facebook_ button on my blog. :sunglasses: