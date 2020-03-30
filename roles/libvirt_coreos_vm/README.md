# Role Name

An ansible role to deploy [Fedora CoreOS](https://getfedora.org/en/coreos/) and [Red Hat CoreOS](https://docs.openshift.com/container-platform/latest/architecture/architecture-rhcos.html) virtual machines into QEMU/KVM.

## Requirements

This is tested on a host running the following
* Ansible 2.9.6
* Fedora 31
* libvirtd 5.6.0

## Role Variables

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

| Variable | Required | Default | Choices | Comments |
|-------------------------|----------|---------|---------------------------|------------------------------------------|
| coreos_type | no | fedora | fedora, redhat | Type of CoreOS host to deploy |
| libvirt_domain_path | no | ${HOME}/documents/vms | | Directory path to deploy virtual machine |
| ignition_path | no | | | Path to ignition 3.0.0 file. One of ignition_config or ignition_path is required. |
| ignition_config | no | | | YAML definition of the ignition 3.0.0 configuration. One of ignition_config or ignition_path is required. |
| vm_name | no | coreos | | Alphanumeric virtual machine name |
| vm_mac | no | 52:54:00:7b:0a:16 | | QEMU/KVM MAC address |
| vm_cpus | no | 2 | | Number of CPU cores |
| vm_memory | no | 2048 | | Amount of memory in MiB |
| vm_disk_size | no | 20G | | Root disk size in GB. Must end in a single 'G' |
| vm_network_name | no | default | | QEMU/KVM network to deploy domain into |
| vm_network_bridge | no | virbr0 | | Host bridge to put virtual machine interface on |
| vm_os_variant | no | fedora31 | | QEMU/KVM OS variant. Get list with ```osinfo-query os``` |
| fcos_stream | no | stable | | Stream to download Fedora CoreOS build from. |
| fcos_build | no | 31.20200310.3.0 | | Fedora CoreOS build number to download |
| rhcos_build | no | 4.3.8 | | Red Hat CoreOS build number to download |

## Example Playbook

```yaml
#!/usr/bin/env ansible-playbook
---
- name: Administer virtual machine
  hosts: localhost
  tasks:
  - name: Deploy Fedora CoreOS virtual machine
    vars:
      coreos_type: fedora
      ignition_config:
        ignition:
          version: 3.0.0
        storage:
          files:
          - path: /etc/sysctl.d/11-lowports.conf
            conetnts:
              source: "data:,net.ipv4.ip_unprivileged_port_start=53"
        systemd:
          units:
          - name: coredns.service
            enabled: true
            contents: "{{ lookup('file', playbook_dir + '/../files/coredns.service') | replace('\n', '\\n') }}"
        passwd:
          users:
          - name: core
            sshAuthorizedKeys:
            - "{{ lookup('file', '/tmp/fcos.pub') }}"
      vm_name: coredns
      vm_mac: 52:54:00:7b:0a:17
      vm_cpus: 2
      vm_memory: 2048
      vm_disk_size: 20G
      vm_network_name: lab
      vm_network_bridge: virbr2
    import_role: 
      name: libvirt_coreos_vm

  - name: Deploy Red Hat CoreOS virtual machine
    vars:
      coreos_type: redhat
      ignition_path: /tmp/openshift/master.ign
      vm_name: ocp-master-1
      vm_mac: 52:54:00:7b:0a:16
      vm_cpus: 4
      vm_memory: 16938
      vm_disk_size: 120G
      vm_network_name: openshift
      vm_network_bridge: virbr0
      vm_os_variant: rhel8.1
    import_role: 
      name: libvirt_coreos_vm
...
```

## License

See [LICENSE](LICENSE)
