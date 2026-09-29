.. include:: ../Includes.txt

.. _introduction:

============
Introduction
============

What does it do?
=================

This extension reads AWS's publicly published `CloudFront edge IP ranges
<https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html>`__ (both
IPv4 and IPv6) and appends them to
:php:`$GLOBALS['TYPO3_CONF_VARS']['SYS']['reverseProxyIP']`, so TYPO3
resolves the real client IP from CloudFront's `X-Forwarded-*` headers instead
of seeing CloudFront's own IP. A PSR-15 middleware does this before TYPO3's
`normalized-params-attribute` middleware reads `reverseProxyIP`. Any
`reverseProxyIP` you already configured is kept and extended, not replaced.

The list is read from AWS on the first request and then stored in TYPO3's
caching framework (registered as the `cloudfront_trusted_proxies` cache) as
compiled PHP — that even lands the list in PHP's opcode cache, so it's read
back faster than going through TYPO3's caching framework alone. Every
following request is served from that cache; only once it's missing or older
than 24 hours is it refetched from AWS. If a refetch fails (AWS unreachable,
bad response, ...), the middleware falls back to the last successfully
fetched list rather than failing the request or dropping proxy trust
entirely.

Why is this needed?
====================

When TYPO3 runs behind AWS CloudFront, every request arrives from one of
CloudFront's edge server IPs, not from the actual visitor. Without telling
TYPO3 to trust those IPs as reverse proxies, TYPO3 can't correctly resolve
the visitor's real IP address from the `X-Forwarded-For` header — which
matters for things like IP-based access restrictions, logging, GeoIP lookups,
or rate limiting. Since AWS regularly adds and removes edge IP ranges, hand
maintaining `reverseProxyIP` isn't practical; this extension keeps that list
current automatically.

Screenshots
===========

This extension has no backend module or user interface — it works entirely
in the background via a request middleware.
