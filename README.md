Proxy Cache Manager
===================

This TYPO3 extension is a flexible and generic way to track the pages that are cached by a reverse proxy like
nginx or varnish, or any useful CDN.

Currently supported backends / providers:
 * Cloudflare CDN
 * Fastly CDN
 * Varnish via curl
 * plain curl

What does it do?
----------------
By embracing TYPO3's Caching Framework this extension provides a new cache to track all pages outputted that are
"cacheable". When an editor changes content on a page, the page cache needs to be cleared - and the CDN / reverse proxy
needs to be informed that the cache is invalid. This is usually done via a HTTP PURGE request to the CDN / proxy server.

The benefits for that are that the editor does not need to worry why out-of-date information is still visible
on his/her website.

Requirements
------------
 * A TYPO3 setup (v9 LTS+) with cacheable content, see the "cacheinfo" extension for what can be tracked with HTTP caches.
 * Either nginx or varnish configured to proxy the TYPO3 system. For nginx, make sure that nginx is compiled to allow
 to flush caches via HTTP `PURGE`. Alternatively, use a CDN like Fastly or Cloudflare for playing around with it.

Setup
-----
Install the extension and make sure to enter the details about your proxy servers, otherwise the default (`IENV:TYPO3_REV_PROXY`) is used.

Don't forget to set the according TYPO3 settings for using proxies, see `$GLOBALS['TYPO3_CONF_VARS']['SYS']['reverseProxyIP']` and `$GLOBALS['TYPO3_CONF_VARS']['SYS']['reverseProxyHeaderMultiValue']`.

Whenever a cacheable frontend page is now called, the full URL is stored in a cache called "tx_proxy" automatically
(a derivative of the Typo3DatabaseBackend cache), tagged with the pageID. Whenever the cache is flushed, the database
table is emptied completely, additionally, a HTTP PURGE call to the reverse proxy is made to empty all caches.
If only a certain tag is removed, the PURGE call is made only to the relevant URLs that are stored in the cache.

### Fastly

Fastly makes a direct HTTP PURGE call to a specific URL, so it needs to be ensured that the request comes directly
from the production environment (CDN backend server) where access to fastly is allowed, and PURGE requests are accepted.

Ensure to set the environment variables `FASTLY_SERVICE_ID` and `FASTLY_API_TOKEN`.

### Cloudflare

Cloudflare v4 API is used to make HTTP requests via Guzzle HTTP and API Tokens. This is done by firing HTTP POST
requests to purge one, multiple or all requests on a domain.

Ensure to set the environment variable `CLOUDFLARE_API_TOKEN` in place.

#### Configuration

Add this to your `AdditionalConfiguration.php` file:

    $GLOBALS['TYPO3_CONF_VARS']['EXTCONF']['proxycachemanager']['cloudflare']['zones'] = [
        'example.com' => 'MY_ZONE_ID'
    ];

By default all administrators can flush the CDN caches via the toolbar
on the top right corner of TYPO3's Backend.

To enable the button for non-admin editors, use this UserTsConfig option:

`options.clearCache.proxy = 1`

To explicitly disable the button for specific administrators, use this
UserTsConfig option:

`options.clearCache.proxy = 0`

#### Limitations
 - Only API Token authentication is available
 - It is not possible to flush by tag (Enterprise Only)

Credits
-------
The extension was created taken all the great from Andreas Gellhaus, Tom Rüther into account, as well
as `moc_varnish` and `cacheinfo` as great work before.

[Find more TYPO3 extensions we have developed](https://b13.com/useful-typo3-extensions-from-b13-to-you) that help us deliver value in client projects. As part of the way we work, we focus on testing and best practices to ensure long-term performance, reliability, and results in all our code.
