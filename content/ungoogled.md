---
title: "ungoogled-chromium"
date: 2021-05-29
tags:
- chrome
- privacy
- security
---

As one of the most used applications in the system, it is ideal to have a secure browser. Browsers have large attack surface due to a combination of their complexity, handling of untrusted input, and use of unsafe languages. Even while browser makers continually work make their browsers more secure, most browsers have poor privacy out of the box. [ungoogled-chromium](https://github.com/Eloston/ungoogled-chromium) is an often recommended browser because of its approach of stripping out Google web services from Chromium.

I have a couple of concerns with this recommendation, namely:

* ungoogled-chromium lags behind upstream Chrome for days [every release](https://github.com/Eloston/ungoogled-chromium/releases).
* The team does not ship official binaries; [third parties contribute prebuilt binaries](https://github.com/Eloston/ungoogled-chromium#downloads), so there is no central party to trust.
* Control Flow Integrity is [disabled by default](https://github.com/Eloston/ungoogled-chromium/commit/7f238719fa0a16e40559df36988aecf0082a8cf3), leaving users vulnerable to control-flow hijacking.
* Component updates are disabled, instead opting to ship components each browser update. Components were originally unbundled from browser updates so that they could be updated faster.
* Links to documentation and troubleshooting are broken by their patches.

Security issues aside, they must be doing unique and great work in privacy, right?

Not necessarily. Most of the functionality of the patches are either in the best case minimally beneficial or can be reproduced with either a setting, a flag, or a switch, and using a browser specifically for these patches should not merit the tradeoff in security.

This blog post aims to detail the patches in the order they apply in, their functions, and how they can be reproduced in Chrome.

### 0001-fix-building-without-safebrowsing.patch, unrar.patch, safe_browsing-disable-incident-reporting.patch, safe_browsing-disable-reporting-of-safebrowsing-over.patch, remove-unused-preferences-fields.patch

These patches disable safe browsing. Related prefs are removed in `remove-unused-preferences-fields`.

* Safe browsing can be turned off in `chrome://settings`.

### 0003-disable-autofill-download-manager.patch
This patch disables form Autofill data transmission to Google.

* Autofill can be turned off in `chrome://settings`.

### 0005-disable-default-extensions.patch
This patch disables
* Google Wallet
* Google Feedback
* Google Web Store
* Google Hangouts
* Google Branding

### 0007-disable-web-resource-service.patch
This patch disables Chrome's WebResourceService, which periodically fetches JSON data from a Google server to dynamically configure the browser. It is(was) used by PluginsResourceService to download security updates for plugins, the last of which was Flash. It is used by PromoResourceService to dynamically change the appearance of the new tab page, although the only platform that still runs promos is iOS.

### 0009-disable-google-ipv6-probes.patch
This patch uses RIPE NCC servers instead of Google servers for IPv6 probes.

### 0015-disable-update-pings.patch
This patch disables pings to clients2.google.com/ for component updates.

* Component updates can be disabled with switch `--disable-component-update`.

### 0017-disable-new-avatar-menu.patch
This cosmetic patch disables the new avatar menu.

### 0021-disable-rlz.patch
This patch disables RLZ, a promotional tag used to measure searches and Chrome usage. This non-unique tag sent is sent with searches made on Google, telemetry, and crash reports.

* RLZ can be disabled by defining `"rlz_disabled":true` in the preferences file.

### disable-crash-reporter.patch
This patch disables the uploading of crash reports to Google. Chromium does not report crashes.

* Disable the crash reporter with switch `--disable-crash-reporter`.

### disable-google-host-detection.patch
This patch disables specific features and restrictions applied to Google domains.

### replace-google-search-engine-with-nosearch.patch
This patch replaces Google with "No Search" (disabling search from omnibox).

* Use another search engine.

### disable-signin.patch, fix-building-without-one-click-signin.patch, disable-gaia.patch
This patch disables browser management of sign-in of Google Accounts. Requires API keys found only in Chrome.

* Disabled with switches `--disable-gaia-services`, `--disable-sync`, `--allow-browser-signin=false`

### toggle-translation-via-switch.patch
This patch disables translation and removes the "Translate to" context menu when `--translate-script-url` is not set.

* Define a non-existent `--translate-script-url`.

### all-add-trk-prefixes-to-possibly-evil-connection.patch, disable-untraceable-urls.patch, block-trk-and-subdomains.patch, disable-webstore-urls.patch, fix-learn-doubleclick-hsts.patch, block-requests.patch
These patches disable all connections hard-coded into the browser using domain substitution. Many of these connections are only made on user interaction and not transparently made in the background.

Connections patched out include
* AppID faucets used in U2F,
* Browser customization,
* Chrome Web Store,
* Component and Extension endpoints,
* Documentation: e.g. the "learn more" page in incognito mode.,
* Extensions functionality,
* Login endpoints,
* Log and crash report endpoints,
* Et cetera.

<details>
    <summary><a href="https://github.com/lynn-stephenson">@lynn-stephenson</a> — you can even change the URLs of "Google URLs" with some switches</summary>
    <p><code>--google-apis-url</code>
    <p><code>--google-base-url</code>
    <p><code>--google-url</code>
    <p><code>--autofill-assistant-url</code>
    <p><code>--autofill-server-url</code>
    <p><code>--cloud-print-url</code>
    <p><code>--connectivity-check-url</code>
    <p><code>--crash-server-url</code>
    <p><code>--cryptauth-http-host</code>
    <p><code>--cryptauth-v2-devicesync-http-host</code>
    <p><code>--cryptauth-v2-enrollment-http-host</code>
    <p><code>--data-reduction-proxy-config-url</code>
    <p><code>--data-reduction-proxy-pingback-url</code>
    <p><code>--data-reduction-proxy-secure-proxy-check-url</code>
    <p><code>--device-management-url</code>
    <p><code>--disable-sync</code>
    <p><code>--disable-sync-types</code>

</details>

### disable-profile-avatar-downloading.patch
This patch disables the downloading of profile avatars from Google.

### disable-gcm.patch
Disables the Google Cloud Messaging component of the `chrome.gcm` API that extensions can use to send messages through Firebase Cloud Messaging.

### disable-domain-reliability.patch
The domain reliability monitor sends info to Google whenever an error occurs while visiting a Google domain.

* Disable domain reliability with switch `--disable-domain-reliability`.

### disable-fonts-googleapis-references.patch
This patch disables references to fonts.googleapis.com/ hardcoded in the browser.

### disable-webrtc-log-uploader.patch
This patch disables the uploading of WebRTC logs for the Hangouts extension.

* Disable reporting additional diagnostics in Hangouts settings.

### use-local-devtools-files.patch
This patch always uses local DevTools files instead of remote files from Google.

### disable-network-time-tracker.patch
This patch disables connections to Google to check if the system time is correct when a website certificate date seems incorrect.

* Disable the network time tracker with switch `--disable-features=NetworkTimeServiceQuerying`

### disable-mei-preload.patch
This patch disables downloading of a list of sites with a high Media Engagement Index, used to determine whether or not a site autoplays.

* Disable MEI preloading with switch `--disable-features=PreloadMediaEngagementData, MediaEngagementBypassAutoplayPolicies`

### fix-building-without-enabling-reporting.patch
This patch disables reporting violations such as COEP.

### disable-fetching-field-trials.patch
This patch disables the downloading of field trials (Google’s A/B testing).

* Field trials can be disabled with switch `--disable-field-trial-config`.

With the relative ease that these settings can be changed, there are few reasons to use `ungoogled-chromium` as a main browser when Chrome can be configured similarly.
