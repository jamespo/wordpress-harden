# wordpress-harden

*Harden your Wordpress Infrastructure, focussing on an installation on Debian Linux.*

## General

- Read the offical guide: https://developer.wordpress.org/advanced-administration/security/hardening/
- Disable file editing in wp-config.php: ```define( 'DISALLOW_FILE_EDIT', true );```

## Local Proxy & Firewall

*Manage & restrict network connections by Wordpress*

- Install a local proxy (eg squid) listening on localhost only.
- Config WP to send all outgoing Wordpress HTTP calls through it, add the below to wp-config.php:

```
define('WP_PROXY_HOST', '127.0.0.1');
define('WP_PROXY_PORT', '3128');
define('WP_PROXY_BYPASS_HOSTS', 'localhost');
```

## PHP Restrictions


## Apache Config
