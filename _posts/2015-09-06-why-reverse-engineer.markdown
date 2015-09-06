---
title: Why reverse engineer mobile apps?
layout: post
date: 2015-09-06
draft: true
---

*This is a post in a small series on reverse engineering mobile apps.*

Mobile apps are the hot thing. Since years. Every business needs an
mobile app, even tiny ones. Sometimes I've the impression, that
building mobile apps became, what was in the early 2000 building
websites. "Hey you're business doesn't have a website, therefore it
isn't future proof. Here, let me build you one." Replace the word
"website" with "mobile app" and you have today's situation.

But this post is not to moan about the status quo. It's about knowing
what's going on with your data. If you think about it, mobile apps is
just a fancy word for "software". Software that runs on your mobile
phone. Software that does stuff with your data. Data like your current
location, your contacts, your photos, and so on.  Most of the time
you'll have a pretty good understanding of what your data is being
used for, but what if there are doubts? In the past there were apps,
that
[stole your address book](https://venturebeat.com/2012/02/14/iphone-address-book/),
in order to provide you with "valuable insights and opportunities". Do
you want this?

The different mobile platforms offer different security models to
handle access to your data. On iOS, for example, the user gets asked
if she wants to give an app access to her address book. On Android,
the user has to confirm, when the app gets installed, whether it has
access to the address book in general. But how does a user know what
then happens with his data?

A local transport providers' app could use the address book data to
calculate routes directly to the home address of my friends. Do do
that, it doesn't need to upload the whole address book to their
servers. The app just needs access to the address book, so that the
user can select which friend they want to visit. On the other hand,
with the next release, the app could change their implementation and
upload every address to their servers. Maybe they want to do this, to
suggest that you visit your friends more often, automatically: "Hey,
it's time to visit Jane again. It's only 15 minutes if you take the
next subway". Wouldn't this be an innovative feature?

The point is that we don't know when and how the apps use our data. If
the app is Open Source Software (or even
[Free Software](https://en.wikipedia.org/wiki/Free_and_open-source_software)),
we can have a look at the source code. But it's not that easy. How do
you know if the source code and the binary you downloaded from the app
store is the same?  Having reproducible builds isn't a trivial
task. The Debian project is working hard on providing reproducible
builds
[for all of their packages](https://wiki.debian.org/ReproducibleBuilds).

The source code of most commercial apps isn't available anyways. In
those cases it's good to have a look at the app from time to time. I
feel it's important to do this, so that companies expect this, fear
the bad press and don't do weird stuff with your data. The more people
know how to do this, the more pressure will be put on the app owners.

In the next post in this series, I'll be looking at the options to
find out what an apps is doing.
