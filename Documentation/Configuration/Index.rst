.. include:: ../Includes.txt

.. _configuration:

=============
Configuration
=============

Enable/disable the extension via the extension configuration (Admin Tools >
Settings > Extension Configuration > `cloudfront_trusted_proxies`), or
directly in :file:`config/system/settings.php`::

	'EXTENSIONS' => [
	    'cloudfront_trusted_proxies' => [
	        'trustEdgeServerIps' => '1',
	    ],
	],

.. container:: table-row

	Property
		trustEdgeServerIps

	Data type
		boolean

	Description
		If enabled, AWS's published CloudFront IP ranges are appended to
		`SYS/reverseProxyIP` on every request. Leave disabled if you don't
		want TYPO3 fetching/caching that IP list, or if you already maintain
		`reverseProxyIP` yourself.

	Default
		1

Clearing the "system" cache group also clears the cached IP list and forces
a refetch on the next request.
