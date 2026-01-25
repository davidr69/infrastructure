# Nautobot


## Installation

### PostGreSQL


```postgresql
CREATE DATABASE nautobot;
CREATE USER nautobot WITH PASSWORD '*****';
GRANT ALL PRIVILEGES ON DATABASE nautobot TO nautobot;
\connect nautobot
GRANT CREATE ON SCHEMA public TO nautobot;
\q
```

## upgrade

```shell
$ pip install --upgrade nautobot
$ pip install -U nautobot-device-onboarding nautobot-golden-config nautobot-dns-models nautobot-plugin-nornir nautobot-capacity-metrics nautobot-ssot

$ nautobot-server collectstatic
$ nautobot-server migrate

$ systemctl restart nautobot nautobot-worker nautobot-scheduler
```
