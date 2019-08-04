---
title: Deploying Yesod with Haskell
date: 2012-07-05
excerpt: How to setup your Yesod project and machine to deploy to Heroku.
---

I finally deployed my Yesod 1.0.0.2 based
[yesod-oauth-demo project](https://github.com/JanAhrens/yesod-oauth-demo)
to Heroku, but it was not the smooth ride I anticipated.

In this post I want to show what
[code changes](https://github.com/JanAhrens/yesod-oauth-demo/commit/d65c105b7e2da659add7990f4f5d0fc8637e3239)
where necessary and what I had to do on my machine and on the Heroku
side to get the first deployment done. It's a longer post and not meant
to be a rant. If you find errors or want to improve this post,
feel free to send me a pull request.

> TL;DR: Get a virtual machine that has the same library versions as
> your Heroku server to build your Yesod application. You have to live
> with the fact that you need to upload large binaries to Heroku, use a
> throw-away git branch for that.

## Catch the qualified import

Each Yesod scaffold generated site includes a Heroku
[Procfile](https://github.com/yesodweb/yesod/blob/master/yesod/scaffold/deploy/Procfile.cg)
that explains the basic steps for the Heroku deployment with commented
lines.

The steps are simple:

1. copy and adjust the `Procfile` from the `deploy/` directory to your
Yesod root directory
2. create a `package.json` file in your Yesod root directory to fake a
node.js application for Heroku
3. add the `heroku` package dependency to your cabal file and adjust
your code
4. build your Yesod project and deploy it to Heroku

The first two steps where straight forward, but I had to make some (in retrospective)
[trivial adjustments](https://github.com/JanAhrens/yesod-oauth-demo/commit/d65c105b7e2da659add7990f4f5d0fc8637e3239)
to the sample code to get the `heroku` package running.
If you are using a Yesod newer release than 1.0.0.2, you have a
[fair chance](https://github.com/yesodweb/yesod/commit/3fc643e0ba58fbd55e9f7bea3a3f0deb85ca21c7)
that you don't have to adjust the code from the Procfile.

The `combineMappings`, `toMapping` and `loadHerokuConfig` functions are
depending on types from the mysterious `AT` module. It's a qualified
import of the `Data.Aeson.Types` module. If you have not done so, cabal
will tell you that you also need to include the `aeson` package in your
cabal file.

I had to do a bit more research to find out that `M.union` is a reference to the
`Data.HashMap.Strict` module, which adds a cabal dependency for the
`unordered-containers` package.

## Shared libraries and old Ubuntu versions

The [deployment guide](https://github.com/yesodweb/yesod/wiki/Deploying-Yesod-Apps-to-Heroku)
from the Yesod wiki, helped me a lot with the following shared library problem:

As you might know, Haskell is a compiled language, which is a good thing. The
bad thing about this is that you have to deploy a compiled binary to Heroku.
This is also not bad per se, but it gets ugly when your system has different
library versions than Heroku. My first deployment attempt failed because
the standard C library (libc) had the wrong version on my system.

The reason for this version mismatch is that I'm using Ubuntu 12.04,
while Heroku is relying on 10.04. Back in 2010 **libc 2.10** was part of the
Ubuntu distribution, but Ubuntu 12.04 comes with **libc 2.15**. I solved
this by using a Ubuntu 10.04 "Lucid Lynx" VirtualBox image with
[Vagrant](http://vagrantup.com/) to compile the binary.

### Vagrant to the rescue

Vagrant is a smart command line wrapper for
[VirtualBox](https://www.virtualbox.org/). The setup is simple:

    $ gem install vagrant
    $ vagrant box add lucid64 http://files.vagrantup.com/lucid64.box
    $ vagrant init lucid64
    $ vagrant up

Having my fresh Ubuntu 10.04 box, I used `vagrant ssh` to get into the
system. There was the next impediment: only ghc6 is supported for this
old Ubuntu version. After rolling my eyes for quite some time, I decided
to install the current
[Haskell Platform](http://hackage.haskell.org/platform/index.html).

If you want to do this, I strongly recommend that you increase the memory of
your Vagrant virtual machine. The whole process is described in detail on the
[Haskell Platform for Linux](http://hackage.haskell.org/platform/linux.html)
site.

To build the executable with the same libc version, make sure that you
do the following steps in your virtual Ubuntu:

    $ cd /vagrant
    $ cabal install
    [...]
    $ cabal clean && cabal configure && cabal build
    [...]
    Linking dist/build/oauth/oauth ...

After you got your binary, you can start deploying or testing.

## Test your build locally

After my first failed deployment to Heroku, I wanted to test the build
locally from within my virtual Ubuntu. First I configured Vagrant to
forward port 8080 from my host system to the VM.

You need to tell your Yesod application what database to use with the
`DATABASE_URL` environment variable (Heroku will do the same). In my
case I choose to use the PostgreSQL database from my host system, which is
reachable from within the virtual Ubuntu via 10.0.2.2.

    $ cd /vagrant
    $ export DATABASE_URL="postgres://[USERNAME]:[PASSWORD]@10.0.2.2:1363/[DATABASE]"
    $ ./dist/build/oauth/oauth Production -p 8080

Yesod will only print output messages if things go wrong. In all other
cases you can start requesting on port 8080.

## Configure Heroku

These steps are only necessary when you want to deploy to Heroku the
first time and do not differ from deploying non-haskell applications.

1. Login to Heroku: `heroku login`
2. Upload you SSH public key: `heroku keys:add ~/.ssh/id_rsa.pub`
3. Create a new Heroku app: `heroku apps:create --stack cedar yesod-oauth-demo`
4. Add a new git-remote: `git remote add heroku git@heroku.com:yesod-oauth-demo.git`

It's important that you use the cedar stack, because it allows the
execution of arbitrary binaries.

## Always check for missing shared libraries

It may happen that your Yesod binary depends on shared libraries, that
are not available on Heroku. You have to check this for each deployment.
If you don't want to risk downtime, you can do this before you deploy
your new release to Heroku.

After checking locally with `ldd ./dist/build/oauth/oauth` for shared
library dependencies on your system (or VM), you have to check if all
dependencies are also available on Heroku. At the moment I'm using
`heroku run bash` to get a shell and search them by hand.

If you found libraries that are not available on Heroku, create a
`libs/` directory on you local system/VM, copy the missing libraries to
that folder and commit + push them with the build Yesod binary to Heroku.

You need to tell Heroku to use that directory by executing
`heroku config:add LD_LIBRARY_PATH=./libs`
the first time that you want to deploy custom binaries. Don't forget to
also add you binary, as described in the next step.

## Ship it

Once you went through the initial trouble of setting up your deployment
machine and adjusting the code, all deployments are straight forward:

1. Create a (throw away) git branch `git checkout -b deploy`
2. Create a binary using `cabal clean && cabal configure && cabal build`
3. Add the binary, commit and push to Heroku master

The reason for the throw away git branch is that Heroku expects you to
push the code using a git branch. The lack of direct Haskell/Yesod
support on Heroku results in committing and pushing a binary, because
Heroku will not compile the code for you. You could use your master git
branch to commit those binaries and push them to Heroku, but this will
result in a large git branch, because each deployment will add a new
large blob to your history.

Remember that you need to build your code in your VM, to have the same
shared library versions as Heroku.

    $ vagrant up
    $ vagrant ssh
    # cd /vagrant
    # git checkout -b deploy
    # cabal clean && cabal configure && cabal build
    # git add -f ./dist/build/oauth/oauth
    # git commit -m "binary"
    # git push -f heroku deploy:master
    # git branch -d deploy

I need to use `git add -f` because I added the `dist/` folder to my
`.gitignore` file.

## That's it

I hope that this was helpful. If you have feedback or want to improve
this post, feel free to use [pull requests](https://github.com/JanAhrens/blog) or write an email.
