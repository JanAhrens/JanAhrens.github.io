---
title: Introducing Yesod quickstart
layout: post
date: 2013-09-17
excerpt: How to setup an up-to-date Yesod and Haskell virtual machine
---

If you want to start with a new technology there is always the challenge of setting up your development environment.
When I started with [Haskell](http://www.haskell.org/) a few years ago I fiddled around with different compiler
versions, [Cabal](http://www.haskell.org/cabal/) and in the end managed to wrack my whole system.
Having learned that lesson I don't recommend installing everything on your local machine when you start to experiment and use a virtual machine instead.

For Haskell and [Yesod](http://www.yesodweb.com/) I put together [Yesod quickstart](https://github.com/JanAhrens/yesod-quickstart.git), a [Vagrant](http://vagrantup.com/)
configuration that'll setup an up-to-date development environment.

Before starting you have to install VirtualBox, Vagrant and the Vagrant Berkshelf plugin.
After all [requirements](https://github.com/JanAhrens/yesod-quickstart#requirements) are installed you can clone the repository and start building your VM.

    $ git clone https://github.com/JanAhrens/yesod-quickstart.git
    $ cd yesod-quickstart/
    $ vagrant up

Depending on your internet connection and your system you have to wait between 30 and 60 minutes until Vagrant has
downloaded and build all packages.

If you survived the initial setup, you're ready to create a Yesod project.
You have to connect to the virtual machine and use the yesod command to scaffold your application.

    $ vagrant ssh
    # cd /vagrant
    # yesod init --bare
    # cabal install && yesod devel

After the Yesod development server is up and running, you can use your default web browser and point it to
[`http://localhost:3000`](http://localhost:3000/). Voilà!

Vagrant will mount the directory containing the
[`Vagrantfile`](https://github.com/JanAhrens/yesod-quickstart/blob/master/Vagrantfile) within the VM under `/vagrant`.
This means that you don't have to get comfortable with `vim` or `emacs` over that SSH connect - just use your favorite
editor on your host system instead. Head over to [the Yesod book](http://www.yesodweb.com/book) to deep dive into Yesod.

I released [Yesod quickstart](https://github.com/JanAhrens/yesod-quickstart) as public domain. Feel free to do whatever you want.
I'm happy about feedback! You can [star the repository](https://github.com/JanAhrens/yesod-quickstart/stargazers),
[suggest improvements](https://github.com/JanAhrens/yesod-quickstart/issues) and/or [send pull requests](https://github.com/JanAhrens/yesod-quickstart/pulls).
