# wordpress-harden

*Harden your Wordpress Infrastructure, focussing on an installation on Debian Linux with
apache, mariadb & php-fpm.*

Dependent on plugins or environment some or all of these recommendations could break your setup,
so test & implement appropriately.

## General

- Read the offical guide: https://developer.wordpress.org/advanced-administration/security/hardening/
- Disable file editing in wp-config.php: `define( 'DISALLOW_FILE_EDIT', true );`

## Local Proxy & Firewall

*Manage & restrict network connections by Wordpress*

- Install a local proxy (eg squid) listening on localhost only.
- Configure WP to send all outgoing Wordpress HTTP calls through it, add the below to wp-config.php:

```
define('WP_PROXY_HOST', '127.0.0.1');
define('WP_PROXY_PORT', '3128');
define('WP_PROXY_BYPASS_HOSTS', 'localhost');
```

Restrict the proxy to only permit allowed endpoints, example `/etc/squid/squid.conf`:

```
# Bind only to the local loopback interface
http_port 127.0.0.1:3128
http_port [::1]:3128

# 1. Define source ACLs for localhost
acl localhost_src src 127.0.0.1/32
acl localhost_src src ::1/128

# 2. Define standard safe and SSL ports
acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 443         # https
acl CONNECT method CONNECT

# 3. Define allowed destination domains
# Note: The leading dot matches wordpress.org and any of its subdomains (e.g., api.wordpress.org)
acl allowed_destinations dstdomain rest.akismet.com
acl allowed_destinations dstdomain .wordpress.org

# --- Access Control Rules ---

# Block traffic to non-standard or dangerous ports
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports

# Allow localhost only to the whitelisted destination domains
http_access allow localhost_src allowed_destinations

# Deny everything else
http_access deny all
```

Configure firewall rules to restrict apache / php to outgoing requests to localhost / any other destinations of your choosing (eg 8.8.8.8 for DNS).

```
iptables -A OUTPUT -m owner --uid-owner www-data -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m owner --uid-owner www-data -d 127.0.0.1 -j ACCEPT
iptables -A OUTPUT -m owner --uid-owner www-data -d 8.8.8.8 -p udp --dport 53 -j ACCEPT
iptables -A OUTPUT -m owner --uid-owner www-data -d 8.8.8.8 -p tcp --dport 53 -j ACCEPT
iptables -A OUTPUT -m owner --uid-owner www-data -j REJECT
```

same for IPv6 if applicable:

```
ip6tables -A OUTPUT -m owner --uid-owner www-data -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
ip6tables -A OUTPUT -d ::1/128 -m owner --uid-owner www-data -j ACCEPT
ip6tables -A OUTPUT -m owner --uid-owner www-data -j REJECT --reject-with icmp6-port-unreachable
```

Use `netfilter-persistent` to save these rules.

## PHP Restrictions

Disable some PHP functions, create `/etc/php/8.2/fpm/conf.d/30-harden.ini` with contents:

```
# list of function to disable globally #
disable_functions =exec,passthru,shell_exec,system,proc_open,popen
```

## Apache Config
