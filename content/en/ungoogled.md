+++
title = "ungoogled-chromium"
date = 2021-05-29
description = "A review of ungoogled-chromium patches"
+++

The [ungoogled-chromium](https://github.com/Eloston/ungoogled-chromium) project
is an often recommended browser to people seeking a private Chromium-based
browser. However, many people are unaware that their patches often regress the
privacy/security rather than improve it.

## Toolchain Hardening

ungoogled-chromium produces some automatic builds through GitHub Actions, as
well as delegating third parties to produce "contributor binaries". Disregarding
the problems with entrusting third parties for builds, all of the builds weaken
the security of the existing mitigations in some form.

[Debian](https://github.com/ungoogled-software/ungoogled-chromium-debian/blob/unified/debian/rules#L59)
fares much worse—it omits building the browser with Clang's
fine-grained, forwards-edge CFI implementation, rendering users vulnerable to
control-flow hijacking. In addition, it uses
[tcmalloc](https://github.com/google/tcmalloc) as the memory allocator, a
performance-oriented allocator without defenses against heap corruption. In
contrast, the default choice (PartitionAlloc) is substantially hardened against
memory corruption and features type-based heap isolation for objects in Blink,
as well as size-based bucketing within the partitions. The build is a
substantial security regression over upstream and shouldn't be recommended to
anyone in good faith.

*   It's worth noting that upstream is
    [removing tcmalloc](https://crrev.com/c/3372825), which is good for users of
    the Debian build, although users would be much safer not using the Debian
    build at all.

Fedora packaging of ungoogled-chromium was historically terrible; it used an
[unsupported toolchain (gcc)](https://github.com/ungoogled-software/ungoogled-chromium-fedora/tree/8300097092afacf6d21d155524c5dc0695a0e43a)
without support for Clang's Control-Flow Integrity, although they have rectified
this as of
[November 10, 2021](https://github.com/ungoogled-software/ungoogled-chromium-fedora/commit/8054d652e66add0c46540296c08d4a593e049063).

In addition, all distros supported by ungoogled-chromium link against system
libraries, which historically aren't compiled with Clang CFI, rendering CFI much
less effective.

## Security Updates

The component updater, responsible for delivering out-of-band security updates
to the various components of the browser, is disabled within ungoogled-chromium.
It's responsible for updating Chrome's CRLSets, which are necessary for
meaningful certificate revocation. Most of the components are delivered via the
component updater because they have a need for out-of-band security updates, and
it's not helpful nor necessary to disable them.

Furthermore, the extensions that users rely on aren't updated automatically,
posing an additional risk to users of the browser.

## WebRTC

A sad thing to note is that ungoogled-chromium chooses to build without mDNS
support, meaning that instead of providing randomized/ephemeral ICE candidates,
the browser provides the user's local IP address. Chrome switched to using mDNS
for WebRTC in August 2019; ungoogled-chromium has disabled mDNS since 2017, as
part of the chrome.mdns API. They have yet to recify this; instead, they are
adopting patches to change the IP handling policy by default, which really
doesn't address the problem in case a user decides to change it from the default
when he's on a network without NAT loopback.

## An Alternative?

Most people should be using Chrome. If one is looking for privacy, disable the
telemetry toggles within `chrome://settings`. Removing every mention of Google
from the codebase is distinct from legitimate privacy/security improvements and
it's evident that the patches cause regressions in security, not improvements.
