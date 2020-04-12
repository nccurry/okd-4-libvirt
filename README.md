# OKD/OCP 4 on Libvirt

These playbooks can be used to install either OpenShift Container Platform (OCP) or Origin Community Distribution of Kubernetes (OKD) into a QEMU/KVM hypervisor using libvirt.

## Assumptions

* The QEMU/KVM host is running Fedora 31+
* All virtual machines will run on a single QEMU/KVM host
* Cluster DHCP will be handled through QEMU/KVM networking
* Cluster DNS and Load Balancing will be handled through a separate Fedora CoreOS "utility" host
 
## Prerequisites

Install ansible and git

Download repository
Initialize git submodule
git submodule init

### Get pull secret

https://cloud.redhat.com/openshift/install/metal/user-provisioned
pull_secret.txt

### Install prerequisites

./playbooks/main.yml -t install_prerequisites -v 

## Miscellaneous

```
# MAC can be generated for KVM using
date +%s | md5sum | head -c 6 | sed -e 's/\([0-9A-Fa-f]\{2\}\)/\1:/g' -e 's/\(.*\):$/\1/' | sed -e 's/^/52:54:00:/'
```

### Useful Commands

```
# Update submodules
git submodule update

# Connect to console
virsh --connect qemu:///system console <name>
```

### Useful Documentation

* https://builds.coreos.fedoraproject.org/browser
* https://origin-release.svc.ci.openshift.org/
* https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/
* https://github.com/coreos/toolbox#toolbox---bring-your-tools-with-you
* https://www.redhat.com/sysadmin/podman-shareable-systemd-services
* https://cbonte.github.io/haproxy-dconv/2.2/configuration.html
* https://www.rapidtables.com/convert/number/octal-to-decimal.html
* https://libvirt.org/formatnetwork.html

