
## nginx

The following was written against nginx stable 1.6.0 ([tarball](http://nginx.org/download/nginx-1.6.0.tar.gz)) (released 2014-04-24, [changelog](http://nginx.org/en/CHANGES-1.6)).

### Installation notes

For complete installation notes, see [the changelog for preparing 0.1.3](../changelog/0.1.3.md).

The primary configuration command we use is:

```bash
./configure --prefix=/etc/nginx \
  --sbin-path=/usr/sbin/nginx \
  --conf-path=/etc/nginx/nginx.conf \
  --error-log-path=/var/log/nginx/error.log \
  --http-log-path=/var/log/nginx/access.log \
  --pid-path=/var/run/nginx.pid \
  --lock-path=/var/run/nginx.lock \
  --user=nginx \
  --group=nginx \
  --with-http_ssl_module \
  --with-http_spdy_module
```

### Contents

Nginx configuration files:

* `nginx.conf` - The main, server-wide nginx config settings.
* `vhosts/default.conf` - A working configuration for a test app serving up a static file, on HTTP and HTTPS (with a self-signed cert).
* `ssl/ssl.rules` - A set of SSL parameters appropriate for a vhost configuration file to `include`. Individual vhosts will still need to use their own `ssl_certificate` and `ssl_certificate_key` parameters, as `vhosts/default.conf` does.

These files expect that the directory structure will be preserved in `/etc/nginx`. In other words, `nginx.conf` should be at `/etc/nginx/nginx.conf`, and `ssl/ssl.rules` should be at `/etc/nginx/ssl/ssl.rules`.

And an additional helper script:

* `init.sh` - An init script for nginx, to allow nginx to start on boot, and to allow control of nginx with the `service` command. Expects certain paths to have been passed to nginx upon `./configure`.

### OCSP Stapling

[OCSP stapling](https://en.wikipedia.org/wiki/OCSP_stapling) &mdash; a method of eliminating 3rd party cert revocation checking &mdash; is not currently enabled, but should be looked into, as it has [performance and privacy benefits](http://blog.cloudflare.com/ocsp-stapling-how-cloudflare-just-made-ssl-30).

An eye should be kept on related standards as they advance -- [OCSP Must-Staple](http://www.ietf.org/mail-archive/web/tls/current/msg10323.html) ([IETF](http://tools.ietf.org/html/draft-hallambaker-tlsfeature-02)) and [OCSP Multi-Stapling](https://casecurity.org/2013/05/07/an-introduction-to-ocsp-multi-stapling/) ([IETF](http://datatracker.ietf.org/doc/rfc6961/)).

### IPv6

Amazon Web Services, 18F's infrastructure provider, does not support IPv6 as publicly addressable IPs for normal instances. They are supported for Elastic Load Balancers (but even then, not inside of Virtual Private Clouds).

Amazon has said (as recently as [April 2014](https://forums.aws.amazon.com/thread.jspa?messageID=536049)) that IPv6 support is in the works for ordinary EC2 instances. Until that actually happens, we're not compiling in the IPv6 nginx module.

### SPDY and HTTP 2.0

We compile in the [SPDY](https://en.wikipedia.org/wiki/SPDY) module for nginx, in addition to the SSL module. SPDY is a protocol, originally designed by Google, to extend HTTP and deliver much better performance, especially for websites with many assets.

As of this writing, we're using nginx 1.6.0, with support for SPDY 3.1. SPDY is being used as the basis for work on [HTTP/2](http://http2.github.io/), which is still very much a draft, and not yet approaching finalization.

We'll update our version and configuration of nginx to accommodate future versions of SPDY (such as 4.0), and when the day comes we'll plan to accommodate HTTP/2.
