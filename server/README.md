# Main server

## network configuration

```yaml
# cat /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  version: 2
  ethernets:
    enp9s0:
      dhcp4: true
      dhcp6: true
    enp6s0:
      addresses:
      - 192.168.1.1/24
      nameservers:
        addresses:
        - 192.168.3.32
        search:
        - lavacro.net
        - lavacro.com
    enp7s0:
      addresses:
      - 192.168.2.1/24
    enp8s0:
      addresses:
      - 192.168.3.1/24
      routes:
      - to: 10.152.183.0/24
        via: 192.168.3.8
      - to: 10.1.42.128
        via: 192.168.3.8
      - to: 172.24.78.0/24
        via: 192.168.3.24
```
