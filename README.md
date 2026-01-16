# Infrastructure #

In the event there is a catastrophic event for any of my infrastructure and/or
services running on them, this is what be required to recreate them.

I have 4 onsite services, and each server is running one or more services:

## server ##



## pi4apps ##



## nube ##



## kube ##
This server only runs MicroK8s and host several apps. Since I run a private
registry on a different server, *containerd* on the Kube server must be
configured to allow non-SSL connections to the private registry. In addition
to installing MicroK8s, modifying the containerd config **MUST** be the next
step since none of the apps will deploy.

The final step is to iterate over the deployment YAML for each application
(obtained from BitBucket) and deploy them.
