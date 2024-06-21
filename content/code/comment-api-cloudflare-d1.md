---
title: "A simple static blog comments system using Cloudflare Workers and D1"
date: 2023-04-12T09:45:41+10:00
draft: false

categories: [Cloudflare, Opine]
tags: [cloudflare d1, cloudflare workers, blog, comments]
toc: false
author: "Nick Perkins"
---
I've spent the last couple of weeks off work, which has given me time to tinker with some different tech. For starters, I migrated this blog over to [Hugo](gohugo.io), a static site generated written in go. I've never had comments on this version of my blog, but I wondered how difficult it might be to roll my own system using [Cloudflare Workers](https://workers.cloudflare.com/)?

This seemed like a good opportunity to play with one of Cloudflare's newest offerings, [D1](https://developers.cloudflare.com/d1/). D1 is Cloudflare's serverless relational database offering built on SQLite. It is still in alpha and isn't ready for production use, but why not take it for a spin?

There has been an [earlier attempt](https://github.com/antoinefink/simple-static-comments) to create a simple blog comment system in Cloudflare workers before, but this used Workers KV for data storage. It doesn't appear to be in use on the creator's blog at this time, and it had some drawbacks, such as taking up to five minutes to update with new comments due to Workers KV limits.

I give you **Opine**.

{{< github repo="nickperkins/opine">}}

Opine is at this point a very simple proof of concept that demonstrates using Cloudflare Workers and D1 to store and fetch comments for a blog. It has no authentication, the input is not sanitised, and there isn't any front end code available at this point. Please, for the love of all that is holy, *do not* use this on a production site.

What it *DOES* have is:

* A D1 database migration to create the `comments` table.
* A simple API created with the [Hono](https://hono.dev/) framework. At a first glance, Hono looks good for this use case and has middleware for CORS, Basic Auth, etc.
* It's written in Typescript, a language I haven't used in a few years.

I'm not sure if I'm going to work on this more or go with another solution for comments on this blog. I think I may look at adding some HTML sanitization to the inputs at the least, and how I might be able to implement a front end and render that using the worker rather. This makes the client side implementation a little easier. In any case, this has been a fun little holiday project.
