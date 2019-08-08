---
title: Hello Signal, Goodbye PGP
date: 2019-08-04
---

After years of maintaining my key I decided to drop PGP and move on to [Signal](https://signal.org). In this post I'll explain why I made this decision.

For a long time I thought that I needed to maintain a PGP key. It was just something that you had to do if you wanted to be serious about your online life.

I went to great lengths to [create the perfect gpg keypair](https://alexcabal.com/creating-the-perfect-gpg-keypair). My master key got stored offline in a secure location and I used a [YubiKey](https://www.yubico.com/products/yubikey-hardware/) to access my private subkey. To establish trust I went to a keysigning party at the [30C3](https://www.ccc.de/en/updates/2013/30c3) and managed to collect 43 signatures.  Every year I went through the process of renewing my key to ensure that if I'd loose access some day, it won't be valid forever.  The next renewal is due later this year and I'm gonna give it a pass.  It's time to say goodbye to my old friend [B911 E6A2 2B4F 3B5F](http://pgp.mit.edu/pks/lookup?op=vindex&search=0xB911E6A22B4F3B5F).

Let's face it: Almost nobody uses PGP. Its adoption rate is very low despite being around since 1991. Maybe it's because it's too complicated, even though tools like [GPG Suite](https://gpgtools.org/) make it very easy. The problem remains that not enough people use it 28 years after its initial release to make a difference. 

Encrypting email isn't a good idea in general. Put all complications with PGP and its adopting rate aside, you can only encrypt email bodies. Still, all metadata is unencrypted. Your provider and other parties can see who is communicating with whom and which email subject is used.

Another problem is the lack of forward secrecy. Once your PGP key gets compromised, all your past communication is also (potentially) compromised. Moxie Marlinspike explains the problem [in a talk](https://vimeo.com/124887048) where he introduces the Signal protocol.

If you want to read more about the many problems of PGP, I can recommend [The PGP Problem](https://latacora.micro.blog/2019/07/16/the-pgp-problem.html).

That's why I decided to use [Signal](https://signal.org) instead. It's an [open-source project](https://github.com/signalapp) that uses modern cryptography (elliptic curve) and provides [forward secrecy](https://signal.org/docs/specifications/doubleratchet/). Signal is available as a mobile app for [Android](https://play.google.com/store/apps/details?id=org.thoughtcrime.securesms) and [iOS](https://apps.apple.com/us/app/signal-private-messenger/id874139669). You can also use Signal on [Linux, Windows and macOS](https://signal.org/download/) - so basically on every platform. You don't need to do anything special to get started. Just download the app, register an account and you're all set. No key generation, no key exchange. Signal trusts the keys of your contact on first use ([TOFU](https://en.wikipedia.org/wiki/Trust_on_first_use)) and alerts you if it changes.

When I migrated to Signal, I wanted to provide everyone with the opportunity to contact me. As it requires a phone number to register, I would have had to publish my private number (I wasn't willing to do that). Luckily, I found a well-written guide that explains how to [use Signal without giving out your phone number](https://theintercept.com/2017/09/28/signal-tutorial-second-phone-number/). In that guide there are a couple of ways listed on how you can obtain an alternative phone number. I ended up using [satellite](https://www.satellite.me/) which is a VoIP app that is only available in Germany. Even though satellite doesn't support text messages, I could still register the account because Signal has a phone call fallback for verification.

I recommend that you give Signal a try and stop worrying about PGP. You can send me your feedback at [+49 156 7856 2789](https://signal.org/download).
