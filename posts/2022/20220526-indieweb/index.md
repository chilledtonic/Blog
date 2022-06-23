---
date: 2022-05-26T07:00:00Z
draft: false
frenchDate: "5 Prairial CCXXX"
tags: ["meta","blog-updates","indieweb"]
title: "I Added Indieweb Support"
tldr: "I added microformat tags to all the blog content, and setup (basic) webmentions. I think it's a fine idea, but don't really think it'll go anywhere."
writer: "Alex S."
aliases: [/220526]
---

I stumbled across [Indie Web](https://indieweb.org/Getting_Started) today, and I think it's a neat idea.

Essentially, it's a set of philosophies and toolsets to allow indie websites to [communicate amongst each other](https://indieweb.org/Webmention), establish a standard for [using your domain as an identity](https://indieweb.org/IndieAuth), and a way for websites to [parse html as rss feeds](https://indieweb.org/h-feed).

Webmention is the most interesting out of all of their various projects, which is essentially a modern replacement for [pingbacks](https://indieweb.org/pingback), if you remember those - I certainly did not.

It's super easy to configure recieving a webmention - [webmention.io](https://webmention.io/) makes it as easy as setting up two "link" tags, and requires no server side code - but sending them is janky as _fuck_.

~~'m not sure if I'll be sending webmentions anytime soon, unless I can write a hugo plugin that does it on build - but the logistics of doing that is honestly such I hassle I probably won't bother.~~

> I used [webmention.app](https://webmention.app/docs#using-ifttt-to-trigger-checks=) to trigger automatic webmemtions via IFTTT whenever this websites RSS feed has a new listing.

I appreciate the idea of getting a ping whenever someone links back to my website, it's neat - but the idea of using webmentions as a full replacement for a full comment system like Disqus makes my head _spin_.

It doesn't help that Indiewebs Wiki is extremely unconcise and difficult to understand.

As for microformat tags, I really don't understand the appeal of them at all. RSS is much easier to generate and parse, but adding the tags to the right spots was trivial, so if it makes someone elses life easier, I really don't mind.

Point being, I spent the afternoon setting it up, only to not fully appreciate the point of all that effort.

# Update

I added webcomments to the blog. It was super easy to setup using [webmention.js](https://github.com/PlaidWeb/webmention.js/); styling is still not quite where I want it to be, but it's hard to see where it fails unless you have a lot of webmentions already - which I don't.

The problem I mainly saw with webmentions was people mistagging them, and therefore they were misrendered - the main example of this was comments hundreds of lines long because they tagged their whole post as a comment, but it was easy to tweak webmention.js to truncate any post over ten words. A short list of recent mentions is what I wanted, not hundreds of comments - and I have that now.

Is it perfect? No. But it's interesting.