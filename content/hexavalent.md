+++
title = "Improving Browser Security"
date = "2021-11-07"
description = "Details about Hexavalent"
summary = "Details about Hexavalent"
+++

[Following up on my Twitter thread](https://twitter.com/CliffMaceyak/status/1459355718834864130) –
I am excited to disclose some details of Hexavalent and discuss potential
future plans.

## About

Hexavalent is a security and privacy focused web browser based on the open
source Chromium project. It seeks to develop meaningful security improvements
with a defined threat model and is currently in development.

## Changes

We've lifted patches from GrapheneOS's
[Vanadium](https://github.com/GrapheneOS/Vanadium) subproject and are currently
in the process of developing and enabling security features that aren't enabled
upstream due to immaturity or performance overhead such as strict origin
isolation, to address real exploits taking advantage of Chrome's site isolation,
and Ubercage, which is V8's heap sandbox intended to prevent memory corruption 
outside the V8 heap.

Hexavalent is able to benefit from upstream hardening done by other 
organizations such as Microsoft to improve the security of the privileged
WebUI by using Trusted Types to eliminate XSS. Our work also involves hardening 
Chrome's heap allocator, PartitionAlloc, to provide more protection against 
heap corruption.

Additionally, the work being done to tighten the Linux sandbox such as by
preventing dynamic code generation by applying restrictions on mapping writable
and executable pages and marking non-executable pages as executable in
processes that do not generate dynamic code is in the process of being tested
and may be upstreamed into the Vanadium subproject.

Some of the work we're doing, such as with the heap allocator, come directly from past work done by other projects such as
[HardenedPartitionAlloc](https://github.com/struct/HardenedPartitionAlloc).

## Future Work

We're going to be looking into reducing the attack surface of the browser by
disabling complex APIs, file formats/parsers, etc, that introduce large amounts
of attack surface and are overly complicated. In addition, we may look into
entirely replacing the heap allocator with one with one focused primarily on
security such as GrapheneOS's
[hardened_malloc](https://github.com/GrapheneOS/hardened_malloc) or Chris
Rolf's [isoalloc](https://github.com/struct/isoalloc).