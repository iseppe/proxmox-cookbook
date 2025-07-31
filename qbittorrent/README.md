## Custom WebUI

[Check VueTorrent UI](https://github.com/VueTorrent/VueTorrent)

A cronjob (root) has been set to automatically pull new changes every Sunday at 23:59

```shell
59 23 * * 0 su - qbtuser -c 'cd /opt/vuetorrent-ui/VueTorrent && git pull'
```
