+++
title = "Some Thoughts About the NSO Group's Pegasus"
date = 2021-07-24
summary = "Recently, a story has been making the rounds about the ‘new’ Pegasus spyware that has lead to mass panic in technical circles that has users worried. There are some questions that users might ask that haven’t been accurately answered and the events have been used by various companies to market their product."
+++

Recently, a story has been making the rounds about the 'new' Pegasus spyware
that has lead to mass panic in technical circles that has users worried. There
are some questions that users might ask that haven't been accurately answered
and the events have been used by various companies to market their product.

Frightened users might ask:

*   Am I at risk?
*   What do I do now?
*   Does this affect the latest Android/iOS?
*   Should I stop using Android/iOS?

The answer is largely **no**.

The Pegasus spyware isn't new. We've known about it for a long time, and we know
it takes advantage of a variety of exploits to exploit different types of
devices. We also know about some specifics of these vulnerabilities, such as the
usage of zero-clicks in iMessage.

Apple has already taken steps to mitigate zero-click vulnerabilities in iMessage,
writing the Blastdoor sandbox, where all untrusted data parsing happens, in
Swift. It has become extremely difficult to exploit iMessage via a zero-click as
of iOS 14, and as such, users should not be worried.

Not that the majority of users should be worried anyway, because they are
**not at risk**, unless they are politically active and suspect someone might
have reason to target them.

Adding onto that, iOS and Android are leagues ahead of any other desktop
operating systems. They both have world class security teams working on them,
and it shows in the cost it takes to write an exploit of such impact. However,
this point is largely moot if your device has stopped receiving updates; if your
vendor is not providing updates then your device should be considered vulnerable
regardless.

If you're shopping for a new device because yours is outdated, buy an iPhone. It
is common consensus among security researchers that an iPhone is one of the most
secure devices you can get, leagues ahead of other phones. However, it isn't
without its downsides, namely in components such as WebKit.

If you're in the Android camp, get a Pixel. Google has world class security
engineers, and Titan M isn't found in any other phones. The
[GrapheneOS](https://grapheneos.org) project is also highly acclaimed by
security researchers and is a hardened OS meant to defend against memory
corruption and other common vulnerabilities. Neither iOS nor GrapheneOS will
protect you from an adversary with unlimited resources and time, but they will
raise the time and the cost to successfully exploit a device.

In the end, Pegasus is not a concern for most users, and you should refrain from
listening to people trying to sell you things in the wake of the news.
