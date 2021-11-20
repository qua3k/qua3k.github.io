+++
title = "Improving Browser Security"
date = "2021-11-07"
description = "Details about Hexavalent"
summary = "Hexavalent is a security and privacy focused web browser based on the open source Chromium project. It seeks to develop meaningful security improvements focusing on mitigating classes of bugs rather than individual vulnerabilities and is currently an unfinished product."
+++

[Following up on my Twitter thread](https://twitter.com/CliffMaceyak/status/1459355718834864130) –
I am excited to disclose some details of Hexavalent and discuss potential
future plans.

## About

[Hexavalent](https://github.com/Hexavalent-Browser/Hexavalent) is a security and
privacy focused web browser based on the open source Chromium project. It seeks
to develop meaningful security improvements focusing on mitigating classes of
bugs rather than individual vulnerabilities and is currently an unfinished
product.

## Our Work

We've adopted platform-agnostic patches from GrapheneOS's
[Vanadium](https://github.com/GrapheneOS/Vanadium) subproject and are planning
to document their functionality in the coming weeks for other developers to
better understand the reasoning behind each change. Some of the hardening is
also being done outside of the browser such as in their hardened bionic libc,
hardened heap allocator, and various toolchain changes which will need to be
evaluated and investigated for inclusion in Hexavalent.

Currently, we're in the process of enabling security features that aren't
enabled upstream due to their needs not being aligned with ours, such as
enabling the "HTTPS-only mode" that can't be trivially enabled upstream due to
breaking compatibity with legacy sites. We've also enabled origin isolation due
to it being the only real mitigation against known speculative execution side
channels that can be used to steal data from sites that aren't cross-origin
isolated. It's something upstream is also looking into enabling by default but
won't until `document.domain` usage drops to a negligible level. The
development of V8's Ubercage is something we're also paying close attention to
and will enable by default once it is more mature.

Some of our current original work involving tightening the Linux sandbox to
prevent dynamic code generation by placing PaX MPROTECT-inspired restrictions on
processes that do not generate dynamic code is currently being tested and may
be upstreamed in the future. We're additionally looking into hardening the heap
allocator based on past work by projects such as
[HardenedPartitionAlloc](https://github.com/struct/HardenedPartitionAlloc). It
is still a work-in-progress but we expect to be able to enable random canaries
in release builds to absorb overflows, zero allocations on free, and more. Some
of the work in
[HardenedPartitionAlloc](https://github.com/struct/HardenedPartitionAlloc) such
as the delayed free implementation won't be re-implemented here as it is
actively being worked on upstream.

Work on redoing the privileged WebUI with a sane Content-Security-Policy won't
be done in Hexavalent either as Microsoft are currently landing patches
upstream using Trusted Types to elimate XSS and any work we do would
effectively be wasted time.

## Future Work

We're going to be looking into reducing the attack surface of the browser by
disabling complex APIs, file formats/parsers, etc, that introduce large amounts
of attack surface and are overly complicated. In addition, we may look into
entirely replacing the heap allocator with one with one focused primarily on
security such as GrapheneOS's
[hardened_malloc](https://github.com/GrapheneOS/hardened_malloc) or Chris
Rolf's [isoalloc](https://github.com/struct/isoalloc).