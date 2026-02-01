# Infrastructure #

In the event there is a catastrophic event for any of my infrastructure and/or
services running on them, this is what be required to recreate them.

I have 4 onsite servers, and each is running one or more services:

## server ##

- iptables
- dhcpd
- postgresql
- HA Proxy

- [LXC](lxc/README.md):
  - jumphost
  - [kafka](kafka/README.md)
  - ftp
  - infrastructure

## pi4apps ##

[Docker](docker)
- redis
- gitea
- registry

## automation host ##

- [Nautobot](nautobot/README.md)
- Docker
  - pi-hole
  - [ceos router and switch images](ceos/README.md)
- SMTP: postfix
- IMAP: dovecot

## nube ##

- nfs
- samba
- Prometheus
- Alertmanager
- Grafana
- bind9
- [MongoDB](mongodb/README.md)
- squid
- [HashiCorp Vault](vault/README.md)


## kube ##

See [kube](kube)
