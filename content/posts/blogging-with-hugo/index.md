---
title: "Blogging with Hugo"
subtitle: or "how to tolerate markdown"
date: 2020-07-28T15:25:48+02:00
draft: false
tags: [blog, hugo, static content]
categories: [personal]
resources:
    - name: featured-image
      src: hugo-logo-spacer-centered.png
images: 
    - hugo-logo-spacer-centered.png
---
# A NEW BLOG

This is not my first blog ever. Over the years I've maintained and scrapped various personal webpages including weblogs.
Now it's time for another try. I want to focus my writing here on tech and IT in whatever one may consider to be the
professional side - but who am I kidding, a blog is never strictly professional. Let's see how this goes. :wink:

## The Tech Stack

I have some prior experience with both maintaining and running [Wordpress](https://wordpress.org/) blogs. So naturally I've
been looking to try something else instead. While static page generators have been around for ages the whole idea seems
to have grown into more than just a valid option by now. I guess we have to thank [github pages](https://pages.github.com/),
[netlify](https://www.netlify.com/) and [AWS](https://aws.amazon.com/) amongst others for this trend.

The cool thing is that running a static web page in 2020 is really no hassle anymore. Gone are the needs to think about
traffic, web servers, security, databases, scripting languages etc. The only thing left to consider is where to host
the page itself and how to rent a domain name to point to the hoster in question.

### The Site Generator

In my hunt for a decent site generator framework / tool I've come upon [Jekyll](https://jekyllrb.com/) a lot. So naturally
this would have been my first tool to try. There were just two major blockers for me.

One is that Jekyll doesn't run readily on all available _OS platforms_. I really need to be able to jump between operating
systems especially when all I'm going to do is edit simple text files.

The second problem with Jekyll was that it doesn't support the text format I've been looking to use: [AsciiDoc](https://asciidoc.org/).
I already know _AsciiDoc_ from documenting various projects in the past and _imho_ it naturally trumps the limited syntax of
[markdown](https://daringfireball.net/projects/markdown/), and that's while being more readable at the same time.

A little more _googling_ around brought [Hugo](https://gohugo.io/) to my attention. It works similarly to Jekyll because
it is still _markdown_ based but it has the possibility to defer to different backends. One of them actually is _AsciiDoc_
via the means of [Asciidoctor](https://asciidoctor.org/). So the choice was obvious to me: _Hugo_ it is.

Little did I anticipate that the current _asciidoctor_ backend is not without its problems. Especially syntax highlighting,
a prime feature of _AsciiDoc_, doesn't seem to integrate with _Hugo_ (or rather its themes) all too well currently:
 - [rgielen.net post about hugo and asciidoctor](https://rgielen.net/posts/2019/creating-a-blog-with-hugo-and-asciidoctor/)
 - [github issue with okta blog](https://github.com/oktadeveloper/okta-blog-hugo/issues/6) 

So the pragmatic decision for now is to start writing in _markdown_ until this becomes a problem. I'm a little bit
concerned about the portability of my blog posts but I'm told there are converters from _markdown_ to _asciidoc_. 

### Hosting

At the time of writing this is all not decided finally yet. I'm currently running a local _Hugo_ instance in dev mode
to try out and configure the site. My goal is to put this on [github](https://github.com/) and have it hosted via
_github pages_. It shouldn't be too complicated to also setup a [github action](https://github.com/features/actions) to
build the site from new pushes on the _main_ branch. Famous last words anyone? :sweat_smile:

### The Theme

Ok, _Hugo_ comes with a lot of themes one could use. I was eying the one [rgielen.net](https://rgielen.net/) uses, it's
called [Coder](https://github.com/luizdepra/hugo-coder/). Should suit me well, right? Nah, but I'm not feeling like a
right out copy cat today. :cat:
 
I also know that light themes are considerd friendly on the eye and overall more professional but I've been choosing
darker themes wherever possible. So this lead me to the [LoveIt](https://hugoloveit.com) theme. It really comes with
a lot of included functionality on top of _Hugo_. Even the syntax highlighting seems good to me. That would be one
further lock-in down the _markdown_ path for me... We will see. :thinking:

I've already been customizing this theme a little bit. The footer needed to be extended for me to be able to fit in 
two links for required legal stuff (thx :de: & :eu:).

```scss
.footer {
  height: 3rem;
}
```
 
And I used all my SCSS knowledge (_think: none_) to put that nice
company logo up there into the header.

```scss
@font-face {
  font-family: 'Infinium Guardian Caps';
  src: url('/fonts/Infinium Guardian Caps.ttf') format('truetype');
}

@font-face {
  font-family: 'Infinium Guardian';
  src: url('/fonts/Infinium Guardian.ttf') format('truetype');
}

$ae-color: #0094FF;

.ae-logo-caps {
  font-family: Infinium Guardian Caps;
  color: $ae-color;
}

.ae-logo {
  font-family: Infinium Guardian;
  color: $ae-color;
}
```

## what about interactive stuff?

Naturally one must wonder, how does a blog work completely static. So there are a few things I've left disabled for now
because they would include pooling functionality from online services like [Disqus](https://disqus.com/).

The following is disabled currently:
 - Search
 - Comments
 - Share Buttons
 - Site counters
 - Google Analytics
 - SEO Stuff
 
 I don't think a lot of that is still necessary today. One can just take a url and paste it e.g. on _Twitter_. I don't
 need to give you a button for this. And when you're on _Twitter_ already why don't use that for comments as well? I'm
 still thinking about what is necessary and what isn't.