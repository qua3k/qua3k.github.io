+++
title = "Email (In)security"
date = 2021-06-01
description = "Email (In)security"
summary = "There is no such thing as secure email. Email is an inherently insecure protocol, conceived at a time when security was an afterthought. There are fundamental flaws with email that cannot be mitigated by slapping encryption on top."
+++

There is no such thing as secure email. Email is an inherently insecure protocol, conceived at a time when security was an afterthought. There are fundamental flaws with email that cannot be mitigated by slapping encryption on top.

## Problems with…

### Email Authentication

[Sender Policy Framework](https://tools.ietf.org/html/rfc7208) is a protocol used to authorize the mail transfer agents allowed to send mail on behalf of a domain. However, SPF alone is almost useless in protecting against spoofing as it is designed to protect against forged senders and not forged headers shown in clients.
* The limitations of SPF are evident in email forwarding, where it fails because the forwarding MTA's IP address in not in the published SPF record.

[DomainKeys Identified Mail](https://tools.ietf.org/html/rfc6376) is a email authentication protocol meant to prove that an email originates from a specific domain by utilizing asymmetric cryptography. The server generates a keypair and publishes the public key in a DNS record. The body and certain email headers are hashed and signed, writing the signature of the hashes as an email header.
* DKIM is limited in its effectiveness in that it also does not prevent forging of email headers, only ensuring the integrity of the content.
* Signatures are also vulnerable to being stripped, and happens in practice because there is no standard way to signal that your emails must be signed unless you deploy DMARC.

[Domain-based Message Authentication, Reporting and Conformance](https://tools.ietf.org/html/rfc7489) solves some of these problems by letting domain owners define a policy for the enforcement of SPF and DKIM and the handling of email authentication failures. DMARC alignment ensures the domain in the `RFC5322.From` address aligns with SPF or the DKIM domain tag and is an essential part of spoofing protection.

### Transport Encryption

Email was not designed with encryption in mind, so when it came time to introduce encryption for message transfer, someone said STARTTLS looked good.
* STARTTLS is [opportunistic encryption](https://blog.filippo.io/the-sad-state-of-smtp-encryption/), meaning an active adversary has the ability to downgrade the connection and force the participants to converse in plaintext, [akin to connecting to a website over `http:` and relying on the server to redirect to `https:`](https://youtube.com/watch?v=MFol6IMbZ7Y).

There are plans to mitigate this, primarily with [MTA-STS](https://tools.ietf.org/html/rfc8461) and [DANE](https://tools.ietf.org/html/rfc7672).
* MTA-STS allows mail servers to advertise their ability to receive TLS connections in a DNS TXT record and serving the policy over `https:`. MTA-STS uses a TOFU model and relies on the security of certificate authorities and DNS.
* DANE has the same end goal, but takes a different approach. DNSSEC can ensure authenticity and integrity of DANE TLSA records so receivers can verify certificates without relying on CAs.

### Email Encryption

[OpenPGP](https://tools.ietf.org/html/rfc4880), the de facto standard for email encryption, has [numerous problems](https://www.whonix.org/wiki/OpenPGP#Issues_with_PGP). Encrypted emails also do not protect metadata such as the sender, recipient, and timestamps, something modern messengers like [Signal](https://signal.org) have done with its [sealed sender](https://signal.org/blog/sealed-sender/).

### Mailbox Encryption

Providers using mailbox encryption can be [legally compelled](https://www.theregister.com/2020/12/08/tutanota_backdoor_court_order/) to disable encryption of new incoming email. Users should not rely on mailbox encryption to conceal the contents of their emails from their provider.

### Conclusion

All of these security mechanisms are optional, and highlights a fundamental flaw in how email services operate. Basic steps such as requiring encryption for all messages will sever communications with entire swaths of providers, and email authentication support is still lackluster on most providers.

Avoid Email.
