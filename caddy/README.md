## Configure the CloudFlare DNS Module
There are a couple ways to do this. xcaddy and direct download of the custom build did not work.
To do this:

```
apt update && apt upgrade

apt install caddy

caddy add-package github.com/caddy-dns/cloudflare
```
