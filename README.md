# cloudfront trusted proxies

TYPO3 extension that trusts AWS CloudFront's published edge server IP ranges as reverse proxies, so TYPO3 resolves the real client IP from CloudFront's `X-Forwarded-*` headers instead of seeing CloudFront's own IP.

## What it does

This extension reads AWS's publicly published [CloudFront edge IP ranges](https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html) (both IPv4 and IPv6) and appends them to `$GLOBALS['TYPO3_CONF_VARS']['SYS']['reverseProxyIP']`, so TYPO3 resolves the real client IP from CloudFront's `X-Forwarded-*` headers instead of seeing CloudFront's own IP. A PSR-15 middleware does this before TYPO3's `normalized-params-attribute` middleware reads `reverseProxyIP`. Any `reverseProxyIP` you already configured is kept and extended, not replaced.

The list is read from AWS on the first request and then stored in TYPO3's caching framework (registered as the `cloudfront_trusted_proxies` cache) as compiled PHP — that even lands the list in PHP's opcode cache, so it's read back faster than going through TYPO3's caching framework alone. Every following request is served from that cache; only once it's missing or older than 24 hours is it refetched from AWS. If a refetch fails (AWS unreachable, bad response, ...), the middleware falls back to the last successfully fetched list rather than failing the request or dropping proxy trust entirely.

## Requirements

- PHP ^8.3
- TYPO3 ^13.0 || ^14.0

## Installation

```bash
composer require mfd/cloudfront-trusted-proxies
```

## Configuration

Enable/disable via the extension configuration (Admin Tools > Settings > Extension Configuration > `cloudfront_trusted_proxies`), or in `config/system/settings.php`:

```php
'EXTENSIONS' => [
    'cloudfront_trusted_proxies' => [
        'trustEdgeServerIps' => '1',
    ],
],
```

| Setting | Default | Description |
| --- | --- | --- |
| `trustEdgeServerIps` | `1` | If enabled, AWS's published CloudFront IP ranges are appended to `SYS/reverseProxyIP` on every request. Leave disabled if you don't want TYPO3 fetching/caching that IP list, or if you already maintain `reverseProxyIP` yourself. |

Clearing the "system" cache group also clears the cached IP list and forces a refetch on the next request.

## Testing

```bash
composer test:unit
```

## License

[GPL-2.0-or-later](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html)

## Maintainer

Maintained by [Marketing Factory Digital GmbH](https://www.marketing-factory.de), written by [Ingo Schmitt](https://www.marketing-factory.de/blog/autoren/ingo-schmitt/).
