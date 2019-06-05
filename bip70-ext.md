---
layout: specification
title: Bip70 User Extension
category: spec
date: 2019-04-24
activation: N/A
version: 0.1
---

Abstract
===============
This document introduces an extension to the Bip70 Payment Protocol which allows end users to take advantage of the security 
offered by the payment protocol and improves the Bitcoin Cash user experience by introducing user friendly payment addresses.

Motivation
===============
Bip70 was first introduced in 2013 as a means to allow merchants to send a Bitcoin address (and various other metadata) signed
by a valid x509 certificate to a customer. This represented a security improvement over the traditional method of sending addresses
in the clear and also improved the user experience because users see "Pay to Example.com" in their wallets rather than a Bitcoin address.

At the time the plans were to find a way to extend Bip70 to end users so that these benefits would acrue to users as well. Unfortunately
years went by without any movement in this area. 

This document tries to finish the uncompleted work.

Specification
===============
We define a payment address format which looks similar to an email address:

`<user>@<domain>`

To send a payment to this type of address a wallet must first internally convert the address into a subdomain and bip70 URI.

Subdomain: `<user>.<domain>`

To enable delegation of the serving the payment requests a wallet must do a DNS lookup for the subdomain's `srv` record. If
such a record exists a Bip70 URI is constructed using the target hostname and port found in the record. Otherwise the URI is
constructed using the <domain> and default port.

Bip70 URI: `bitcoincash:?r=<domain>:<port>/r/<user>`

A GET request is made to the URI endpoint following the standard Bip70 protocol and the returned x509 domain in the returned x509
certificate is valided to match the address' subdomain (in addition to standard x509 validation). 

The wallet can now build a transaction paying the requested outputs. If the payment request provided a payment URI the wallet can
follow the standard protocol and push the payment transaction to the payment endpoint. Otherwise it must just broadcast the payment.

Third Party Subdomains
==================
The above protcol is very simple and works well in the cases where the user owns the domain. However, most users are probably not expected
to purchase domains. In this case subdomains can be created and allocated by third parties with the end user creating the certificate locally
and passing it to the service provider to have them get it signed by a CA and returned. The user then creates and signs their payment request and
uploads it to the server which will host it for them.

From a security perspective third party subdomains should be considered less secure than owning the domain outright as the domain owner
could create a new certificate for the subdomain at any time and start serving up payment requests which appear valid but contain a different
address. A saving grace in this case, however, is the user would have cryptographic proof that the server is acting maliciously (valid signed payment requests
for the same subdomain with different certificates). Any malicious behavior on the part of the server could be detected and annouced publicly.

Bitcoin Cash Specific CAs
==========================
A possible way to keep service providers honest is to establish Bitcoin Cash specific certificate authorities for this use case.

Such an authority would behave differently than a traditional CA in the following ways:

1. They would accept certificates without an expiration (Most CAs require very short expiration like 90 which is both cumbersome on the user
and easier to attack as the service provider could just wait for the cert expiration and then issue the subdomain to someone else).

2. They would never issue duplicate certificates unless the user explictly revoked their cert. This would prevent the service provider from
swapping out payment requests assuming the CA is honest.

3. They could issue certs for an unlimited number of subdomains (most CAs have pretty severe rate limits). 

The downsides of this approach is it's not clear there is anyone in the Bitcoin Cash community with a strong enough repuation to run a CA and
such CAs would need to be hardcoded into all wallets. 

Reusable Address
=================
While this specification could work with standard Bitcoin Cash addresses it would obviously be much better to avoid address reuse. Therefore it is
recommend this payment scheme only be used with a reusable address type (defined elsewhere). 





 

