---
title: Why reverse engineer mobile apps?
layout: post
date: 2015-09-07
excerpt: In the first post of my "reverse engineering mobile apps" series, I'm arguing why you should learn how to reverse engineer apps.
---

*This post is part of a series on reverse engineering mobile apps.*

Mobile apps are in demand - for several years now. Every business
needs an app, even tiny ones. Sometimes I'm under the impression that
building mobile apps became, what was in the early 2000s, building
websites. "Hey, your business doesn't have a website, therefore it isn't
future proof. Here, let me build (and sell) you one." Replace the word
"website" with "mobile app" and you have today's situation.

Nevertheless this post isn't to moan about the status quo. It's about
knowing what's going on with your data. If you think about it, "mobile
app" is just a fancy word for "software". Software, that runs on your
mobile phone. Software, that messes with your data. Data like your
current location, your contacts, your photos, and so on.  Most of the
time you'll have a pretty good understanding of what your data is
being used for, but what if there are doubts? In the past, there were
apps, that
[stole your address book](https://venturebeat.com/2012/02/14/iphone-address-book/),
in order to provide you with "valuable insights and opportunities". Do
you want this?

The different mobile platforms offer different security models to
manage the access to your data. On iOS, for example, the user is asked
if she wants to give an app access to her address book. When a user
installs an app on Android, he has to confirm, whether it gets access
to his address book. There is no choice left. Either you give access,
or you can't install the app at all. No matter how an app got access
to the data, how will a user know what happens with it next?

Image a local transport providers' app, that gives you the directions
to the homes of your friends. In order to simplify the process, the
app uses the address book data to now where your friends live. It's
not necessary to upload the whole address book to build this
feature. The user selects a friend's address and the app sends this
particular information to the server, in order to give directions. On
the other hand, with the next update, the local transport provider
could change their implementation and upload every address to their
servers. Maybe they want to do this, to suggest that you visit your
friends more often - automatically: "Hey, it's time to visit Jane
again. It's only 15 minutes if you take the next subway". Wouldn't
this be an innovative feature?

The point I want to make is that we don't know when and how apps use
our data. If the app is Open Source Software (or even
[Free Software](https://en.wikipedia.org/wiki/Free_and_open-source_software)),
we can have a look at its source code. Unfortunately, it's not that
easy. How do you know that the source code was used to build the
binary, you downloaded from the app store?
[Reproducible builds](https://wiki.debian.org/ReproducibleBuilds/About)
could help, but providing them isn't a trivial task. Debian is working
hard to make
[every binary package reproducible](https://reproducible.debian.net/reproducible.html),
but they're still not there, yet. I'm not aware that there's a similar
project for mobile apps.

The source code of most commercial apps isn't available anyway. In
those cases it's good to have a closer look at the app's behavior from
time to time. I feel it's important to do so, to raise the companies
awareness, make them fear bad press, and keep them from messing around
with your data. The more people know about how to do this, the more
pressure will be put on the app owners.

In the next post of this series, I'll be looking at your options to
find out what an app is doing.
