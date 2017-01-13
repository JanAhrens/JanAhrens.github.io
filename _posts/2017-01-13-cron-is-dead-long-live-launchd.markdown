---
title: cron is dead, long live launchd!
layout: post
date: 2017-01-13
---

Now that I finally created my [tarsnap](https://tarsnap.com) backup script, how
do I execute it regularly? Oh, I know: My Mac is just an Unix system, I'll use
cron!

At least that's what I thought I'll do. After a few attempts to get cron to do
the job, I learned that there's a better way on macOS:
[launchd](https://en.wikipedia.org/wiki/Launchd).

launchd does a lot more than executing scripts cron-style. Like
[systemd](https://en.wikipedia.org/wiki/Systemd) on Linux, launchd is a
replacement for a lot of old school Unix tools, like cron, inetd, init,
[etc](https://en.wikipedia.org/wiki/Launchd#History).

At it's core, launchd distincts daemons and agents. Dameons are processes that
always run in the background, while agents describe regular jobs that are to be
executed on certain events. There are a lot of different events to choose from.
For example you can trigger an agent, when a device gets mounted, when a file
gets created, or when a certain time arrives.

What really helped me in learning how to write my first launchd agent was
[launchd.info](http://www.launchd.info/). Unlike the [Apple
documentation](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html),
it contains useful snippets and concise explanations. I highly recommend that
you also have a look at the launchd agents that some of your applications put into
`~/Library/LaunchAgents`.

Below you can see the agent that I ended up creating. You can learn how to
load/unload agents and about the meaning of the different options at
[launchd.info](http://www.launchd.info/).

If you're testing your script and you don't want to wait for the next hour to
arrive, you can start it immediately with `launchctl start
eu.jan-ahrens.tarsnap`.

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
	    <key>Label</key>
	    <string>eu.jan-ahrens.tarsnap</string>
		<key>EnvironmentVariables</key>
		<dict>
			<key>PATH</key>
			<string>/bin:/usr/bin:/usr/local/bin</string>
		</dict>
	    <key>ProgramArguments</key>
	    <array>
		<string>/bin/bash</string>
		<string>/Users/jan/bin/run-tarsnap-backup</string>
	    </array>
	    <key>StartInterval</key>
	    <integer>3600</integer>
	    <key>StandardOutPath</key>
	    <string>/Users/jan/.tarsnap.log</string>
	    <key>StandardErrorPath</key>
	    <string>/Users/jan/.tarsnap.log</string>
	    <key>KeepAlive</key>
	    <dict>
		<key>NetworkState</key>
		<true/>
	     </dict>
	    <key>ExitTimeout</key>
	    <integer>900</integer>
	    <key>Nice</key>
	    <integer>10</integer>
	</dict>
	</plist>

P.S.: cron itself is implemented as a launchd daemon. You can find it at `/System/Library/LaunchDaemons/com.vix.cron.plist`.
