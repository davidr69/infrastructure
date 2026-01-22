# Docker

For both pi4apps and automation host:

```shell
/etc/docker# cat daemon.json
{
    "dns":["192.168.3.32"]
}
```

### Registry cleanup

cron:
`0       3       *       *       0`

```shell
cid=`docker ps | grep registry | awk '{print $1}'` && docker exec $cid registry garbage-collect /etc/docker/registry/config.yml
```
