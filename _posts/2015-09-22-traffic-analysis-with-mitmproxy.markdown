---
title: Analyse app traffic with mitmproxy
layout: post
date: 2015-09-22
excerpt: In the second post of my &quot;reverse engineering mobile apps&quot; series, I give an introduction on how to analyse the traffic of apps.
---

*This post is part of a series on reverse engineering mobile apps.*

1. [Why reverse engineer mobile apps?](/2015/09/07/why-reverse-engineer.html)
2. **Analyse app traffic with mitmproxy**

---

When you want to inspect an app's behavior, the first step is to look
at the traffic it produces. In this post I'll show you how to do this
with "mitmproxy".

Fortunately, most apps encrypt their traffic nowadays. This means that
you will have a hard time using a regular sniffer like
[tcpdump](http://www.tcpdump.org/) or
[Wireshark](https://www.wireshark.org/). Instead you have to do a
[man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)
to see what traffic the app produces. I'm using
[mitmproxy](https://mitmproxy.org/doc/mitmproxy.html) for this.

After you [installed](https://mitmproxy.org/doc/install.html) and started mitmproxy on your computer, you need
to configure your phone to send its traffic through your computer.

On Android, this can be done by connecting both devices with the same
Wi-Fi. Your computer needs to serve as a proxy server for your phone.
This can be configured under
"Settings&nbsp;&rarr;&nbsp;Wi-Fi". Press long on the network you're
connected to, select "Advanced options" and put your computer's IP
address as "Proxy hostname" and the mitmproxy port as "Proxy port"
(the default setting is 8080).

[![](/assets/android-proxy-small.png)](/assets/android-proxy.png)

To setup mitmproxy for other platforms, have a look at [the
documentation](http://docs.mitmproxy.org/en/latest/modes.html).

Once your app sends its traffic through your computer, you need to get
your phone to trust the mitmproxy certificate authority. This process
has to be repeated every time you change the CA in mitmproxy. For
Android, iOS and Windows phone, this is very easy. Just open the
browser on your phone, visit [mitm.it](http://mitm.it), select your
platform and that's it.

Now you can open the app you want to inspect and see the unencrypted
traffic in mitmproxy on your computer.

If you don't see traffic popping up or the app is not getting any
data, this might have various reasons. Before assuming that the MITM
attack failed, you should keep in mind that the data might be still
cached on the device. For example, try "pull-to-refresh" or closing and
reopening the app to force it to get fresh data.

It's possible that the app doesn't respect the proxy settings on the
phone and thus escapes the MITM attack. This would be the case if the
app still gets data and you don't see activity in mitmproxy. You can
use
[one of the other techniques](http://docs.mitmproxy.org/en/latest/modes.html)
to intercept the traffic, when you suspect that the setting
isn't respected.

Sometimes app developers configure their app not to trust the certificate
authorities provided by the mobile phone. They bundle
the correct certificate or certificate authority with their app. This
technique is called
[Certificate Pinning](https://www.owasp.org/index.php/Certificate_and_Public_Key_Pinning).
In those cases it's not possible to inspect the traffic even if the
traffic gets routed through your computer. The app simply rejects the
fake certificate authority and refuses to connect to its servers. You can
spot those cases when neither the app nor mitmproxy get any data. If we want to continue our analysis, we have to go one step further and replace the pinned
certificate or certificate authority inside the app.

It's also possible that the app isn't using HTTP(S) to get its
data. I came across such a situation when I tried to inspect the traffic of the
[Threema app](/2014/03/22/threema-protocol-analysis.html).

Threema opens a regular socket and implements its own
protocol. Prebuild MITM tools can't handle those situations because
they know nothing about the protocol. Even though the apps traffic
reaches the app, it can't MITM attack it. Once again, simple traffic
analysis doesn't help here.

Although, traffic analysis has it's limitations I still recommend it
as a first step. It's simple to setup and a lot of apps can be
analysed in a short amount of time.

---

*In the next post I will show you how to learn more about an app by
looking at it's binary. This post will focus purely on Android apps.*
