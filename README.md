# OKD/OCP 4 on Libvirt

These playbooks can be used to install either OpenShift Container Platform (OCP) or Origin Community Distribution of Kubernetes (OKD) into a QEMU/KVM hypervisor using libvirt.

## Assumptions

* The QEMU/KVM host is running Fedora 31+
* The QEMU/KVM host has 32GB+ Memory and a modern multi-core CPU
* All virtual machines will run on a single QEMU/KVM host
* Cluster DHCP will be handled through QEMU/KVM networking
* Cluster DNS and Load Balancing will be handled through a separate Fedora CoreOS "utility" host (via CoreDNS and HAProxy)
 
## Prerequisites

### Configure host prerequisites

```
# Install pip and git
sudo dnf -y install python3-pip git

# Install ansible 
pip3 install ansible --user

# Ensure default local pip bin directory is on PATH
echo "export PATH=$PATH:~/.local/bin" >> ~/.bashrc
source ~/.bashrc

# Verify ansible is installed
ansible --version

# Clone repository
git clone git@github.com:nccurry/okd-4-libvirt.git

# Initialize submodules
git submodule update --remote
```

### Get pull secret

Go to [cloud.redhat.com](https://cloud.redhat.com/openshift/install/metal/user-provisioned) and download your pull secret

Put it where ansible can find it and modify the ```pull_secret_path``` variable in [defaults/main.yml](defaults/main.yml).

### Fill out variables in [defaults/main.yml](defaults/main.yml)

## Deploy cluster

```
./playbooks/main.yml -v
```

## Teardown cluster
```
./playbooks/main.yml -v -e teardown=true
```

## Miscellaneous

```
# MAC can be generated for KVM using
date +%s | md5sum | head -c 6 | sed -e 's/\([0-9A-Fa-f]\{2\}\)/\1:/g' -e 's/\(.*\):$/\1/' | sed -e 's/^/52:54:00:/'
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

