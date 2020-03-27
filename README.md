# OKD 4 on Libvirt

## Prerequisites

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
# Connect to console
virsh --connect qemu:///system console <name>
```

### Useful Documentation

* https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/
* https://github.com/coreos/toolbox#toolbox---bring-your-tools-with-you
* https://www.redhat.com/sysadmin/podman-shareable-systemd-services
* https://cbonte.github.io/haproxy-dconv/2.2/configuration.html
* https://www.rapidtables.com/convert/number/octal-to-decimal.html
* https://libvirt.org/formatnetwork.html

