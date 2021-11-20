+++
title = "ungoogled-chromium"
date = 2021-05-29
description = "A review of ungoogled-chromium patches"
+++

The browser is one of the most used applications in any system. For many,
significant portions of their online life flow through their browser and as
such, the browser and the developers behind it are in a position to abuse their
power to invade peoples' privacy.

As a consequence, projects such as
[ungoogled-chromium](https://github.com/Eloston/ungoogled-chromium) were born to
"remove integration with Google services" within the codebase and "enhance
privacy, control, and transparency".

However, there are some glaring flaws with the ungoogled-chromium project. For
instance, ungoogled-chromium lags behind upstream Chrome for
[days](https://chromereleases.googleblog.com/)
[every release](https://github.com/Eloston/ungoogled-chromium/releases). The
time that n-days are exploitable in ungoogled-chromium is *significantly* longer
than in either Chrome or Edge.

The team also **does not** ship official binaries and requires users to either
trust an additional unknown
[third party](https://github.com/Eloston/ungoogled-chromium#downloads) or build
from source. It *may not* be a concern depending on the vendor distributing the
binaries, however, and
[reproducible builds don't solve this problem](https://blog.cmpxchg8b.com/2020/07/you-dont-need-reproducible-builds.html).

Keeping that in mind, the third parties that are trusted to distribute prebuilt
binaries tend to cripple security features either by weaken exploit mitigations
such as Control-Flow Integrity by
[opting to use system libraries](https://github.com/ungoogled-software/ungoogled-chromium-archlinux/blob/7c5d8f35f827d317a003f89534e9df33bc791080/PKGBUILD#L96)
often not compiled with CFI or
[building with unsupported toolchains](https://github.com/ungoogled-software/ungoogled-chromium-fedora).

Furthermore, component updates are disabled, even though components are
delivered via the component updater because they require faster out-of-band
updates than regular browser updates, and the ungoogled-chromium team does not
seem to have any interest in delivering component updates.

Finally, *the patches don't make sense*. Performing domain substitution on every
URL in the codebase, including documentation and troubleshooting, doesn't appear
to be driven by a real or perceived threat.

In addition, most of the functionality of the patches can be easily driven by
**disabling crash and metrics reporting in Chrome settings**. The patches that
can't be reproduced easily in Chrome settings are in the best case minimally
beneficial or actually harmful. Using a browser specifically to gain a perceived
notion of privacy should not involve making significant tradeoffs in security.

As such, this blog post is meant to shed some light on the function of
ungoogled-chromium patches and how they compare to Chrome's user facing
settings.

{{< table_of_contents >}}

## Safe Browsing

The ungoogled-chromium project has written and uses multiple patches to
wipe Google's Safe Browsing service from the codebase even though users can
disable Safe Browsing from the settings. For instance:

### fix building without safebrowsing

removes parts of the Safe Browsing service such as the Safe Browsing
interstitial.

### unrar

disables "support for safe browsing inspection of rar files".

### disable incident reporting

disables "the safebrowsing incident reporting where you could upload
information about a blocked URL to Google (also added a trk prefix to the URL so
we get notified if this happens again in the future)."

### disable reporting of safebrowsing override

disables "reporting of the safebrowsing override, i.e. the report sent if a
user decides to visit a page that was flagged as "insecure". This
prevents trk:148 (phishing) and trk:149 (malware)".

### remove unused preferences fields

removes "unused Safe Browsing and Sign-in fields from the Preferences file".

## Autofill

The ungoogled-chromium project removes parts of the Autofill service to prevent
Chrome from sending
"[hashed descriptions of the form and its fields](https://chromium.googlesource.com/chromium/src/+/refs/tags/97.0.4686.3/components/autofill/core/browser/autofill_download_manager.cc#254)"
when the user encounters a web form. Note that Autofill can be turned off in
Chrome settings.

## Default Extensions

The ungoogled-chromium project disables certain built-in extensions such as the
Chrome Web Store, and the exclusive to `GOOGLE_CHROME_BRANDING` in-app payments
support application and Feedback app.

## WebResourceService

ungoogled-chromium removes portions of Chrome's WebResourceService, used by
Chrome's
[PluginsResourceService](https://chromium.googlesource.com/chromium/src/+/refs/tags/97.0.4686.3/chrome/browser/plugins/plugins_resource_service.cc#79)
to download security updates to various pepper plugins in Chrome, such as
[Adobe Flash](https://chromium.googlesource.com/chromium/src/+/refs/tags/97.0.4686.3/chrome/browser/resources/plugin_metadata/plugins_win.json#3)
and the
[Chrome PDF viewer](https://chromium.googlesource.com/chromium/src/+/refs/tags/97.0.4686.3/chrome/browser/resources/plugin_metadata/plugins_win.json#23);
this may change with the advent of the
[Unseasoned PDF viewer](https://docs.google.com/document/d/1olIb-1IFVqP2fLTq0eAdW-aL-K2dDDZGDe5mZgHGfO8/edit)
and the deprecation of PPAPI.

## IPv6 Probes

ungoogled-chromium uses RIPE NCC servers instead of Google servers for IPv6
probes (connectivity checks) and adds a feature flag for disabling IPv6 probes
altogether.

## Component Updates

The ungoogled-chromium project stubs out the URL for component updates, posing a
legitimate risk by denying users out-of-band security updates to components such
as Chrome's CRLSets, used to
"[quickly block certificates in emergency situations](https://dev.chromium.org/Home/chromium-security/crlsets)".
In contrast, GrapheneOS's Vanadium subproject
[disables potentially invasive component updater pings](https://github.com/GrapheneOS/Vanadium/blob/bef7ed9b6884968e7506badea79a13a3247485f2/patches/0048-disable-component-updater-pings-by-default.patch)
while still allowing for component updates, and
[they are looking into providing component updates via their own server](https://github.com/GrapheneOS/Vanadium/issues/62).

## Sign-in / new avatar menu

ungoogled-chromium disables sign-in – it used to additionally control the new
avatar menu but doesn't anymore and never got renamed.

## RLZ

ungoogled-chromium disables the use of RLZ, a promotional tag used to measure
searches and Chrome usage. The non-unique tag
"[contains information about how Chrome was obtained, the week when Chrome was installed, and the week when the first search was performed](https://www.google.com/chrome/privacy/whitepaper.html#measurepromotions)"
and is sent with searches made on Google, telemetry, and crash reports. However,
users can "opt-out of sending this data to Google by...installing a version
downloaded directly from [www.google.com/chrome](https://www.google.com/chrome).
To opt-out of sending the RLZ string in Chrome OS, press Ctrl + Alt + T to open
the `crosh` shell, type `rlz disable` followed by the enter key, and then
reboot your device."

## Crash Reporting

ungoogled-chromium disables the uploading of crash reports to Google. However,
builds without `GOOGLE_CHROME_BRANDING`
[do not report crashes](https://chromium.googlesource.com/chromium/src/+/refs/tags/97.0.4686.3/components/crash/core/app/crash_reporter_client.cc#26).

## Google Domains Functionality

ungoogled-chromium disables specific features and restrictions applied to
Google domains.

## Search Engine

ungoogled-chromium replaces Google with "No Search" (disabling search from
omnibox).

## Google Accounts (GAIA)

ungoogled-chromium disables browser management of sign-in of Google Accounts.

## Translate

ungoogled-chromium disables translation and removes the "Translate to" context
menu when `--translate-script-url` is not set.

## Domain Substitution

ungoogled-chromium includes multiple patches to disable all connections
hard-coded into the browser using domain substitution. Many of these
connections are only made on user interaction and not transparently made in the
background.

Connections substituted out include

*   AppID faucets used in U2F,
*   Browser customization,
*   Chrome Web Store,
*   Component and Extension endpoints,
*   Documentation: e.g. the "learn more" page in incognito mode.,
*   Extensions functionality,
*   Login endpoints,
*   Log and crash report endpoints,
*   Et cetera.

## disable-profile-avatar-downloading.patch

ungoogled-chromium disables the downloading of high-res profile avatars from
Google whenever a user selects an avatar in Chrome settings.

## chrome.gcm

ungoogled-chromium removes the Google Cloud Messaging component of the
`chrome.gcm` API that extensions can use to send messages through Firebase
Cloud Messaging.

## Domain Reliability

The Domain Reliability monitor sends info to Google whenever an error occurs
while visiting a Google domain. The ungoogled-chromium project disables the
creating of the Domain Reliability service in all cases, even though it can be
disabled by
[disabling metrics and crash reporting](https://chromium.googlesource.com/chromium/src/+/refs/tags/97.0.4686.3/chrome/browser/domain_reliability/service_factory.cc#35).

## Remote Fonts

ungoogled-chromium removes references to
[fonts.googleapis.com/](https://fonts.googleapis.com/) hardcoded in the
browser, replacing them with alternative local fonts.

## WebRTC Log Uploader

ungoogled-chromium disables the uploading of WebRTC logs for the private WebRTC
logging APIs used by the Hangouts/Meets services, although it would mean no data
would have been uploaded in the first place due to domain substitution.

## Local DevTools Files

ungoogled-chromium always uses local DevTools files instead of remote files
from Google.

## Browser Network Time

ungoogled-chromium disables
"[occasional queries to a Google server to retrieve an accurate timestamp](https://chromeenterprise.google/policies/#BrowserNetworkTimeQueriesEnabled)".

* Disable the network time tracker with switch
`--disable-features=NetworkTimeServiceQuerying`

## MEI Preloading

ungoogled-chromium disables downloading of a
[list of sites](https://chromium.googlesource.com/chromium/src/+/refs/tags/97.0.4686.3/chrome/browser/resources/media/mei_preload/preloaded_data.pb)
with a high
[Media Engagement Index](https://developer.chrome.com/blog/autoplay/#media-engagement-index)
used to determine whether or not a site autoplays. Users can disable MEI
preloading with switch
`--disable-features=PreloadMediaEngagementData,
MediaEngagementBypassAutoplayPolicies`

## Reporting API

ungoogled-chromium allows building with `enable_reporting = false`. (disables
Reporting API)

## Field Trials

ungoogled-chromium disables the downloading of field trials (Google’s A/B
testing).

## Conclusion

ungoogled-chromium really isn't a security-oriented browser and is often a
regression over Chrome due to various problems with their approach. There are
other downstream forks such as Vanadium that add meaningful security
improvements that upstream won't due to performance regressions, but Chrome
will always be the first to receive security updates, which is kind of a big
thing going for it.

## See also

*   [Hexavalent](https://github.com/Hexavalent-Browser/Hexavalent)
*   [Vanadium](https://github.com/GrapheneOS/Vanadium)
*   [Google Chrome Privacy Whitepaper](https://www.google.com/chrome/privacy/whitepaper.html)
*   [Random switches the Chromium Authors expose](https://gist.github.com/qua3k/6bde83af45355f8b3e7eef845a8a2ece)
