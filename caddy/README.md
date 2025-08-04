## Configure the CloudFlare DNS Module
There are a couple ways to do this. xcaddy and direct download of the custom build did not work.
To do this:

```
apt update && apt upgrade

apt install caddy

caddy add-package github.com/caddy-dns/cloudflare
```

**NOTE: After every upgrade, you need to re-add the cloudflare package**

## Enable Caddy admin API on all interfaces

**1. Edit the Caddyfile**

```shell
# /etc/caddy/Caddyfile
# Global block. Needs to be above anything else.
{
  "admin": {
    "listen": "0.0.0.0:2019"
}

# All other configs
[...]
```

**2. Restart caddy.service and reload Caddyfile**

```shell
systemctl restart caddy

caddy reload --config /etc/caddy/Caddyfile
```

If you don't do this, you can't reload the configuration using `caddy reload`

[See Caddy docs reference](https://caddyserver.com/docs/caddyfile/options#admin)
