# OKD/OCP 4 on Libvirt

These playbooks can be used to install either OpenShift Container Platform (OCP) or Origin Community Distribution of Kubernetes (OKD) into a QEMU/KVM hypervisor using libvirt.

They are useful to set up an OKD/OCP lab on some spare hardware and to demonstrate the utility of deploying containers using systemd on Fedora CoreOS.

## Assumptions

* The QEMU/KVM host is running Fedora 31+
* The QEMU/KVM host has 32GB+ Memory and a modern multi-core CPU
* All virtual machines will run on a single QEMU/KVM host
* Cluster DHCP will be handled through QEMU/KVM networking
* Cluster DNS and Load Balancing will be handled through a separate Fedora CoreOS "utility" host (via CoreDNS and HAProxy)
* The user ansible is using has sudo permissions on the QEMU/KVM host
 
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

```
# Fill out parameters as you see fit
vi defaults/main.yml
```

## Deploy cluster

```
# Execute playbook
./playbooks/deploy.yml -v

# Wait for bootstrap process to complete
# Once this finishes you can delete the bootstrap vm to save resources
okd-install wait-for bootstrap-complete --dir ~/path/to/ignition --log-level debug
# or
openshift-install wait-for bootstrap-complete --dir ~/path/to/ignition --log-level debug

# Wait for installation process to complete
okd-install wait-for install-complete --dir ~/path/to/ignition --log-level debug
# or
openshift-install wait-for install-complete --dir ~/path/to/ignition --log-level debug
```

The cluster uses the public ssh key in ```~/.ssh/id_rsa.pub``` for ssh authentication so you can log in using that key and the core user.

Entries are created in ```/etc/hosts``` to allow you to access the default cluster routes.
You can reach the console via: 
```https://console-openshift-console.apps.<cluster_name>.<domain_suffix>```

For example:
```https://console-openshift-console.apps.ocp.lab.local```

You can get the kubeadmin credentials via:

```
# Location of your ignition files
ignition_directory=~/ocp/ocp
cat ${ignition_directory}/auth/kubeadmin-password
```

You can log using oc in via:

```
# Location of your ignition files
ignition_directory=~/ocp/ocp

# Value for cluster_name
cluster_name=ocp

# Value for network.domain_suffix
base_domain=lab.local

# Login
oc login \
  --insecure-skip-tls-verify=true \
  -u kubeadmin \
  -p $(cat ${ignition_directory}/auth/kubeadmin-password) \
  https://api.${cluster_name}.${base_domain}:6443 
```

## Teardown cluster and cleanup all files
```
./playbooks/teardown.yml -v
```

## Known Issues 

### Failed to get "write" lock
Often when attempting to start the virtual machines the [virt](https://docs.ansible.com/ansible/latest/modules/virt_module.html) ansible module will fail with

> internal error: process exited while connecting to monitor: 2020-04-12T17:43:32.040107Z qemu-system-x86_64: -drive file=/home/user/documents/vms/ocp-bootstrap.lab.local.qcow2,format=qcow2,if=none,id=drive-virtio-disk0: Failed to get "write" lock
> Is another process using the image [/home/user/documents/vms/ocp-bootstrap.lab.local.qcow2]?

You can work around this issue by starting the virtual machines manually from within virt-manager.

## Useful Links

* Latest Fedora CoreOS Builds
  * https://builds.coreos.fedoraproject.org/browser
* Fedora CoreOS Documentation
  * https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/
* Latest Origin Community Distribution of Kubernetes Builds
  * https://origin-release.svc.ci.openshift.org/
* Latest OpenShift Container Platform Builds
  * https://openshift-release.svc.ci.openshift.org/
  * https://mirror.openshift.com/pub/openshift-v4/clients/ocp/
* Latest Red Hat CoreOS builds
  * https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/
* CoreOS Toolbox
  * https://github.com/coreos/toolbox#toolbox---bring-your-tools-with-you
* Running podman containers with systemd
  * https://www.redhat.com/sysadmin/podman-shareable-systemd-services
* HAProxy configuration
  * https://cbonte.github.io/haproxy-dconv/2.2/configuration.html
* Convert octal numbers to decimal (useful for ignition file permissions)
  * https://www.rapidtables.com/convert/number/octal-to-decimal.html
* Libvirt Network XML definition
  * https://libvirt.org/formatnetwork.html

