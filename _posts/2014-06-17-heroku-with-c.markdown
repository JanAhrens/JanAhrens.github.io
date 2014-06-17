---
title: "Heroku with C"
layout: post
date: 2014-06-17
---

Whenever I want to know my external IP address, I use
[ifconfig.me](http://ifconfig.me). It's a small service that returns
your IP address along with some additional information. The only
downside is that it's relatively slow. In this post I'll describe how
I implemented a subset of ipconfig.me in C and deployed it to
[Heroku](https://www.heroku.com/). TL;DR: Try
[ipconfig.herokuapp.com](http://ipconfig.herokuapp.com/).

* [Is it possible?](#possible)
* [How Heroku works](#heroku)
* [Returning the IP address](#code)
* [Deployment](#deployment)
  * [Hijacking Node.js](#hijack-node)
  * [Procfile](#procfile)
  * [Pushing the code](#push-it)
* [What is it good for?](#summary)
  * [The numbers](#numbers)

# Is it possible? <a name="possible"></a>

When I first got the idea I didn't knew if Heroku would be able to run C
applications. I remembered that I managed to run a
[Haskell application on Heroku](http://blog.jan-ahrens.eu/2012/07/05/yesod-deployment.html)
a while ago. Officially Heroku neither supports Haskell nor C. It
simply doesn't know how to prepare the code and do things like
dependency management.

For my Haskell project I was able to resolve this by compiling a
binary on my machine and deploying it directly to Heroku. Heroku was then
able to execute the binary and thus serve the web-pages. I decided to
use this method with my C application, again.

# How Heroku works <a name="heroku"></a>

In order to know how I would build my idea, I had to read a bit of
the well-written and extensive
[Heroku documentation](https://devcenter.heroku.com/categories/reference).

If you draw a rough picture, Heroku consists of two components: An
intelligent load-balancer and one or more application "servers", also
known as [dynos](https://devcenter.heroku.com/articles/dynos). The
load-balancer maps a DNS name, like `foobar.herokuapp.com`, to one or
more dynos.

Each application server (or dyno) has to run a HTTP server, that receives
requests from the and responds to them.

# Returning the IP address <a name="code"></a>

In addition to routing the requests, the Heroku load-balancer will
also append
[some headers](https://devcenter.heroku.com/articles/http-routing#heroku-headers)
before passing the request to the application. The `X-Forwarded-For`
header contains the IP address of the client that does the
request. This is the external IP address of the client and the only
information that my application should return.

The [resulting application](https://github.com/JanAhrens/ipconfig-http-server/blob/master/main.c) is very simple:

1. Start a HTTP server
2. Wait for requestsp
3. Extract the IP address from the `X-Forwarded-For` header
4. Return the address
5. Goto 2.

To minimize the effort I implemented the
[application single-threaded](https://github.com/JanAhrens/ipconfig-http-server/blob/master/main.c)
and managed to only use 135 lines of C code. I think this is not bad
if you consider that it also includes a minimal HTTP server 😃.

You can test it on your local machine by
executing the binary (you might have to re-compile it using `make`). The only thing that you have to keep in mind
is that you have to simulate the Heroku load-balancer by sending the
`X-Forwared-For` header on your own:

    ./main 8080
    curl --header 'X-Forwarded-For: 8.8.8.8' http://127.0.0.1:8080/

After creating the binary, my next step was to deploy it to Heroku.

# Heroku Deployment <a name="deployment"></a>

What I really like about Heroku is how easy it is to deploy your
application. All you need to know is how to use git.
When you do a `git push`, Heroku tries to determine what language
your project uses. It then takes the matching
[buildpack](https://devcenter.heroku.com/articles/buildpacks) to
install dependencies, compile the code and run the application.
Heroku comes with a number of pre-configured buildpacks for languages
like Java, Ruby, Node.js and so on.

## Hijacking Node.js <a name="hijack-node"></a>

For my Haskell project I hijacked the existing
[Node.js buildpack](https://github.com/heroku/heroku-buildpack-nodejs).
I learned this trick from the
[Yesod wiki](https://github.com/yesodweb/yesod/wiki/Deploying-Yesod-Apps-to-Heroku)
during my attempt to deploy the Haskell application. The idea behind this approach
is relatively simple: Because you already have a binary, the buildpack doesn't need to
do anything, so you have to find a buildpack that can easily be fooled into doing nothing.

For the Node.js buildpack this works perfectly: You only have to
create a
[`package.json`](https://github.com/JanAhrens/ipconfig-http-server/blob/master/package.json)
file, state that there are no dependencies, give it a fake-name and a
fake-version. That's it!

<script src="http://gist-it.appspot.com/github/JanAhrens/ipconfig-http-server/blob/master/package.json"></script>

The reason why this works, is that the Node.js buildpack has a simple
[`detect` mechanism](https://github.com/heroku/heroku-buildpack-nodejs/blob/master/bin/detect)
that only checks for the `package.json` file. With the detect script,
Heroku verifies that the buildpack is responsible for the current project.

## Procfile <a name="procfile"></a>

After I had tricked the Node.js buildpack into accepting the code, the
last missing piece was the `Procfile`.  In the Procfile you state what
kind of processes your application provides. In my case it's simple,
because it's only a web-server in form of the binary.

<script src="http://gist-it.appspot.com/github/JanAhrens/ipconfig-http-server/blob/master/Procfile"></script>

## Pushing the code <a name="push-it"></a>

The disadvantage of shipping binaries is that you have to compile them before you deploy.
In contrast to more conventional Heroku application, you have to run `make`, add the binary to git and then push the code:

    make
    git add main
    git commit -m "new release"
    git push heroku master

You can play with the application on Heroku. I was able to secure the
domain `ipconfig.herokuapp.com`. You can either [open it in your browser](http://ipconfig.herokuapp.com/) or use your favorite terminal program:

    curl http://ipconfig.herokuapp.com/

If the application isn't used in a while, Heroku will shutdown
it's dyno and has to start a new one for you. In my experience this
takes around 40 seconds, so don't be surprised if your first request
is way too slow.

# What is it good for? <a name="summary"></a>

If you compare the time invested, with the achieved functionality, I'd
say there is nothing to discuss. Writing programs in C is
slow. But on the other hand it was interesting to learn all those
details about Heroku.

I did this small project just for fun. Nevertheless it's interesting
to know how well it "performs". Since it's a single-threaded
application I did three tests: 100 requests in sequence, 10 requests
in parallel and 100 requests in parallel.

The Heroku load-balancer is able to queue requests until the dyno is
available again. That's the reason why the response time is increasing, if
more requests are executed in parallel. You could solve this issue
by starting more dynos, but that would cost a lot of money.

## The numbers <a name="numbers"></a>

*"I only believe in statistics that I doctored myself" -- Winston Churchill*

For my benchmark I used [ApacheBench](https://httpd.apache.org/docs/2.2/programs/ab.html).
Luckily `ab` has a switch that produces HTML tables, so here are the hard numbers:

### 100 requests in sequence

<table class="table">
<tr><th>&nbsp;</th> <th>min</th>   <th>avg</th>   <th>max</th></tr>
<tr><th>Connect:</th><td>   48</td><td>   52</td><td>   69</td></tr>
<tr><th>Processing:</th><td>   55</td><td>   58</td><td>   65</td></tr>
<tr><th>Total:</th><td>  103</td><td>  110</td><td>  134</td></tr>
</table>

    ab -n 100 http://ipconfig.herokuapp.com/


### 100 requests, 10 in parallel

<table class="table">
<tr><th>&nbsp;</th> <th>min</th>   <th>avg</th>   <th>max</th></tr>
<tr><th>Connect:</th><td>   49</td><td>   54</td><td>   65</td></tr>
<tr><th>Processing:</th><td>   54</td><td>   60</td><td>   63</td></tr>
<tr><th>Total:</th><td>  103</td><td>  114</td><td>  128</td></tr>
</table>

    ab -c 10 -n 100 http://ipconfig.herokuapp.com/

### 100 requests in parallel

<table class="table">
<tr><th>&nbsp;</th> <th>min</th>   <th>avg</th>   <th>max</th></tr>
<tr><th>Connect:</th><td>   58</td><td>  142</td><td>  237</td></tr>
<tr><th>Processing:</th><td>  186</td><td>  188</td><td>  156</td></tr>
<tr><th>Total:</th><td>  244</td><td>  330</td><td>  393</td></tr>
</table>

    ab -c 100 -n 100 http://ipconfig.herokuapp.com/

I hope you enjoyed this experiment and I'd love to hear your
thoughts. You can find [my PGP key](https://keybase.io/janahrens) at
Keybase.io.
