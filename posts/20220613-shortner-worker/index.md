---
title: "Permashortlinks & Cloudflare Webworkers"
date: 2022-06-13T07:00:00Z
frenchDate: "24 Prairial CCXXX"
draft: false
tldr: "In the spirit of Indieweb, I created a permashortlink parser - and a traditional shortlink creator - using CloudFlare Webworkers."
tags: ["development","JavaScript","indieweb"]
aliases: [/220613]
---

I've been investigating more deeply about [Indieweb](https://indieweb.org/permashortlink), namely their idea of a permashortlink - put succinctly, it is inevitable that shortlinking providers will either
1. Fold, or
2. Deprecate underused links.

This means that relying on them - and they are awfully convenient - is a net negative to the longevity of any web content you produce - and we've [already seen it happen](https://www.poynter.org/reporting-editing/2009/tr-ims-shutdown-shows-risk-of-relying-on-free-services-to-drive-web-traffic/).

The Indieweb solution to this deprecation problem is, of course - to roll your own. Reading the wiki about this raises the same problem I have with a lot of Indieweb content - it's a wiki clearly written by developers. I don't think it's actionable for the average web content consumer - even if they were gunhoe about running their own blog - to roll any of the permashortlink solutions mentioned on the [wiki](http://tantek.pbworks.com/w/page/21743973/Whistle) by themselves.

But that's not really the point of this article - I am a developer, I find the idea of permashortlinks appealing - so I wrote my own parser for them.

# How Permashortlinks Work
The idea of a permashortlink is that if I control the pathing of the content, then I can create a shortlink to that content that doesn't have to be stored in a database.

For example, most blogs organize their content by the date you posted them. You could, say, have your Hugo config [explicitly create](https://gohugo.io/content-management/urls/#permalinks) all your blogposts with the following pathing:

```yaml
permalinks: {
  posts: '/:year/:month/:day/'
}
  
```

Lets say that creates a url like `cannedfi.sh/2022/06/13`.

The permashortlink of that url could hypothetically be `cannedfi.sh/220613` - to unravel that back to the full url, you would write some sort of [serverside script](http://tantek.pbworks.com/w/page/21743973/Whistle) that lenghtens that url back to its original permalink.

_However,_ Hugo has a concept called [Aliases](https://gohugo.io/content-management/urls/#aliases). These let you setup secondary links that redirect to the same content very easily, so in my case I didn't have to write _any code at all_ - I just had to apply this feature to the idea of permashortlinks and now when I write a new blogpost I simply add the following to it's config:

```yaml
---
aliases: [/220613]
---
```

# Making them even shorter
My use case for permashortlinks is primarily using them on [the fediverse](https://luckycat.chat) or Twitter - as well as pasting them in the bottom of slide decks (via [permashortcitation](https://indieweb.org/permashortcitation)). That last usecase especially benefits from as short an URL as possible - so I decided to setup a secondary url - [cfsh.lol](https://cfsh.lol)

You could likely setup a second shorter url and just use CNAME dns records to redirect to the longer url, but this may break pathing if you decide to use the url for other things - so I setup a [CloudFlare Worker.](https://workers.cloudflare.com/) I'm not going to go extensively into how workers work in this article, but I will link to a simple tutorial later on in this article.

Put short, It's a Javascript Router that redirects from the shortlink to the full link; the code looks like this

```js
router.get('/b/:slug', async request => {
	let link = "https://cannedfi.sh/"+request.params.slug;
	if (link) {
		return new Response(null, {
			headers: { Location: link },
			status: 301,
		});
	} else {
		return new Response('Key not found', {
		status: 404,
		});
	}
});
```

It might seem a little heavy handed to use a worker for something so simple - but it'll all come together in this next part.

# Linking to stuff I don't control
Permashortlinks work for everything that _I create on this blog_. However, Ocassionally I need to shorten links to someone elses content - for that you'd need a traditional, good old fashoned, [keys to values](https://developers.cloudflare.com/workers/learning/how-kv-works/) data store.

Is this against the spirit of a permashortlink - maybe. My argument is that I control the database of shortlinks, and it in the grand scheme of things it will remain - ironically - fairly short, as I don't need to shorten URLs very often.

This code is based off of [this tutorial](https://dev.to/mmascioni/build-a-link-shortener-with-cloudflare-workers-1j3i). However, the code in said tutorial fails to take into account checking for any sort of url collisions - it is unlikely the random slug will collide, but it is very likely I may accidentally shorten the same URL twice, and that kind of duplicity I cannot abide.

I chose instead of generating a random slug to instead Hash the URL and truncate the hash. I'm not very worried about collisions at this point in time, but I'll let you know if this decision bites me in the ass on a future date.

```js
router.post('/n', async request => {
  let requestBody = await request.json();
  let slug = ADLER32.str(requestBody.url).toString(16).slice(0,5)
  if ('url' in requestBody) {
    if ('exp' in requestBody) {
      await shorten.put(slug, requestBody.url, { expirationTtl: 8600 });
    }else {
      await shorten.put(slug, requestBody.url);
    }
    let shortenedURL = `${new URL(request.url).origin}/l/${slug}`;
    let responseBody = {
      message: 'Link shortened successfully',
      slug,
      shortened: shortenedURL,
    };
    return new Response(JSON.stringify(responseBody), {
      headers: { 'content-type': 'application/json' },
      status: 200,
    });
  }else {
    return new Response("Must provide valid URL", {status: 400});
  }
})
```

Somewhere someone is angry I'm using [ADLER32](https://en.wikipedia.org/wiki/Adler-32) as a hash function - but that's why I'm doing it! To make you, imaginary reader, explicitly upset about it.