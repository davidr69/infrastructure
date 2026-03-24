# registry

## configuration


## cleanup

```shell
cid=`docker ps | grep registry | awk '{print $1}'` && docker exec $cid registry garbage-collect /etc/docker/registry/config.yml
```
