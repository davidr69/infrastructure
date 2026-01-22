# LXC


- jumphost
- kafka
- ftp
- infrastructure



```shell
lxc launch -p default ubuntu:22.04 jumphost --config limits.cpu=1 --config limits.memory=1GiB
lxc network attach lxdbr0 jumphost eth0 eth0
lxc config device set jumphost eth0 ipv4.address 192.168.32.8

lxc launch -p default ubuntu:22.04 kafka --config limits.cpu=2 --config limits.memory=2GiB
lxc network attach lxdbr0 kafka eth0 eth0
lxc config device set kafka eth0 ipv4.address 192.168.32.20
```

advertised.listeners=PLAINTEXT://kafka.lavacro.net:9092

To make `htop` work:

```text
# crontab -e
@reboot mount -t tmpfs tmpfs /proc/spl/kstat/zfs
```



lxc network attach lxdbr0 ftp eth0
lxc config device set ftp eth0 ipv4.address 192.168.32.8
lxc config set ftp security.privileged true
lxc config device add ftp reolink disk source=/mnt/reolink path=/mnt/reolink



systemctl daemon-reload



lxc config set jumphost image.description="ubuntu 24.04 LTS amd64 (upgraded)"
lxc config set jumphost image.release="noble"
lxc config set jumphost image.version="24.04"

lxc config set kafka image.description="ubuntu 24.04 LTS amd64 (upgraded)"
lxc config set kafka image.release="noble"
lxc config set kafka image.version="24.04"


lxc launch -p default ubuntu:24.04 infra --vm --config limits.cpu=2 --config limits.memory=4GiB


