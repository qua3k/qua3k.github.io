+++
title = "ungoogled-chromium"
date = 2021-05-29
description = "ungoogled-chromium 补丁的评论"
+++

抱歉，我的英文比我的中文好，有的词可能会翻译错的。

许多人经常推荐 [ungoogled-chromium](https://github.com/Eloston/ungoogled-chromium) 项目作为
Chrome 的私人替代品。然而，许多人不知道他们的补丁通常会降低浏览器的隐私/安全性，而不是改善它。

## 工具链强化

ungoogled-chromium produces some automatic builds through GitHub Actions, as
well as delegating third parties to produce "contributor binaries". Disregarding
the problems with entrusting third parties for builds, all of the builds weaken
the security of the existing mitigations in some form.

[Debian](https://github.com/ungoogled-software/ungoogled-chromium-debian/blob/unified/debian/rules#L59)
的情况很糟糕--他们省略了使用 Clang 的控制流完整性 (CFI) 实现，使用户容易受到控制流劫持。此外，他们使用
[tcmalloc](https://github.com/google/tcmalloc) 作为堆分配器。tcmalloc 是一个面向性能的分配器，
没有堆保护。

The default choice (PartitionAlloc) is substantially hardened against memory
corruption and features type-based heap isolation for objects in Blink, as well
as size-based bucketing within the partitions. The build is a substantial
regression over upstream and shouldn't be recommended to anyone in good faith.

*   It's worth noting that upstream is
    [removing tcmalloc](https://crrev.com/c/3372825), which is good for users of
    the Debian build, although users would be much safer not using the Debian
    build at all.

Fedora packaging of ungoogled-chromium was historically terrible; it used an
[unsupported toolchain (gcc)](https://github.com/ungoogled-software/ungoogled-chromium-fedora/tree/8300097092afacf6d21d155524c5dc0695a0e43a)
without support for Clang 的 CFI, although they have rectified
this as of
[November 10, 2021](https://github.com/ungoogled-software/ungoogled-chromium-fedora/commit/8054d652e66add0c46540296c08d4a593e049063).

In addition, all distros supported by ungoogled-chromium link against system
libraries, which historically aren't compiled with Clang CFI, rendering CFI much
less effective.

## 安全更新

The component updater, responsible for delivering out-of-band security updates
to the various components of the browser, is disabled within ungoogled-chromium.
It's responsible for updating Chrome's CRLSets, which are necessary for
meaningful certificate revocation. Most of the components are delivered via the
component updater because they have a need for out-of-band security updates, and
it's not helpful nor necessary to disable them.

Furthermore, the extensions that users rely on aren't updated automatically,
posing an additional risk to users of the browser.

## An Alternative?

Most people should be using Chrome. If one is looking for privacy, disable the
telemetry toggles within `chrome://settings`. Removing every mention of Google
from the codebase is distinct from legitimate privacy/security improvements and
it's evident that the patches cause regressions in security, not improvements.
