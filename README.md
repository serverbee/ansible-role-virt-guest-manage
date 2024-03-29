## Virt guest manage role

This role uses [official ansible virt module](https://docs.ansible.com/ansible/2.9/modules/virt_module.html) for managing
RHEL based virtual machines with using [KVM](http://www.linux-kvm.org) as a hypervisor. By default it uses AlmaLinux fork of RHEL.
You can control all your VMs directly via settings in group_vars or host_vars.
The main feature of this role is the parallel installation of all virtual machines which are set in `virt_guest_list`.

#### To begin

##### Supported distro versions at the virtual machine level

* CentOS el6 (not tested with latest role versions)
* CentOS el7
* AlmaLinux el8
* AlmaLinux el9

##### Update default values for your needs

First of all you have to update `virt_guest_init_passwd` with setting a custom password.
Second thing is you have to find and set the nearest mirror for downloading all components of RHEL based installator.

##### Out of the scope of this role

This role can manage LVM based virtual disks for VMs but you have to create Physical Volume (PVS) and Volume Group (VGS) before using it.
Also there are two different options for virtual network interfaces of VMs. The first way is to use an existed and already configured Linux bridge
and the second one is to use [libvirt networking](https://wiki.libvirt.org/page/Networking). Anyway the last option can be managed with
`virt_network_list` directly from this role.


#### Variables

##### General

* `virt_guest_dependency_qemu_kvm_role`: [optional, default `true`]: Whether enable a dependency role for qemu-kvm
* `virt_guest_mirror`: [default: `http://repo.almalinux.org`]: Almalinux mirror what will be used to install OS
* `virt_guest_os_location`: [default: `{{ virt_guest_mirror }}/almalinux/9/BaseOS/x86_64/os`]: Location path where Almalinux os componnets stored
* `virt_guest_kickstart_config_dir`: [default: `/tmp/kickstart`]: The path where kickstart files with be created
* `virt_guest_kickstart_config_port`: [default: `8000`]: The port for downloading kickstart configuration durring an installation
* `virt_guest_kickstart_config_url`: [default: `ip of a Hypervisor`]: The url for downloading kickstart configuration durring an installation
* `virt_guest_kickstart_config_serve_timeout`: [default: `90`]: Time in seconds to serve kickstart files
* `virt_guest_kickstart_installation_timeout`: [default: `480`]: Time in seconds to finish a kickstart installation
* `virt_guest_init_passwd`: [required]: A password of root user which you will use for the fisrt login
* `virt_guest_init_hostname`: [default: `fresh-installed.local`]: The first hostname on virtual machine OS level

##### Virt network list and it own settings

* `virt_network_list`: [required, default: `{}`]: Virt network declarations
* `virt_network_list.key`: [required]: The name of virtual network (e.g. `br-nat0:`)
* `virt_network_list.key.router`: [required]: The ip address of virtual router (e.g. `192.168.2.1`)
* `virt_network_list.key.netmask`: [required]: The netmask of virtual network (e.g. `255.255.255.0`)
* `virt_network_list.key.start`: [required]: The first ip of the virtual network range (e.g. `192.168.2.1`)
* `virt_network_list.key.end`: [required]: The last one ip of the virtual network range (e.g. `192.168.2.254`)

##### Virt guest list and it own settings

* `virt_guest_list`: [required, default: `{}`]: Virt guest declarations
* `virt_guest_list.key`: [required]: The name of virtual machine (e.g. `example-vm:`)
* `virt_guest_list.key.autostart`: [optional, default `true`]: Whether enable an autostart for virtual machine
* `virt_guest_list.key.uuid`: [required]: Universally unique identifier of virtual machine (e.g. `ad852ffe-07d9-43ec-9f5a-ced644f9a7a5`)
* `virt_guest_list.key.cpu`: [optional, default `1`]: Set CPU core limits
* `virt_guest_list.key.ram`: [optional, default `2`]: Set RAM limits. This value set in GiB
* `virt_guest_list.key.disk`: [required]: Virt guest disk(s) declarations
* `virt_guest_list.key.disk.[s|v]d[a-z]`: [required]: The name of virtual disk in virtual machine (e.g. `sda`, `sdb`, `sdc`, etc or `vda`, `vdb`, `vdc`, etc )
* `virt_guest_list.key.disk.[s|v]d[a-z].type`: [optional, default `block`]: The type of virtual disk (e.g. `block`, `file` )
* `virt_guest_list.key.disk.[s|v]d[a-z].name`: [required only for `.type: file`]: The name of virtual qemu disk file
* `virt_guest_list.key.disk.[s|v]d[a-z].format`: [optional, default `raw`]: The qemu format type of virtual disk (e.g. `raw`, `qcow2`)
* `virt_guest_list.key.disk.[s|v]d[a-z].format_options`: [optional, default `preallocation=off`]: The qemu disk format options (e.g. `preallocation=metadata`)
* `virt_guest_list.key.disk.[s|v]d[a-z].vg`: [required only for `.type: block`]: The name of LVM Volume Group
* `virt_guest_list.key.disk.[s|v]d[a-z].lv`: [required only for `.type: block`]: The name of LVM Logic Volume
* `virt_guest_list.key.disk.[s|v]d[a-z].size`: [required, except using physical drives]: Size of virtual disk (e.g. `2048M`,`10G`, `1T`, also `20%VG` or any equivalent can be used for LVM based disks)
* `virt_guest_list.key.disk.[s|v]d[a-z].device`: [required only for physical drives]: The full path to the physical drive on a Hypervisor (e.g. `/dev/sdb`, `/dev/nvme0n1`, etc)
* `virt_guest_list.key.disk.[s|v]da.fstype`: [optional, default `xfs`]: Filesystem type for all partitions inside a virtual disk (e.g. `ext4`, `ext3`, etc)
* `virt_guest_list.key.network`: [required]: Virt guest network(s) declarations
* `virt_guest_list.key.network.eth[0-9]`: [required]: The name of network interface in virtual machine (e.g. `eth0`, `eth1`, etc )
* `virt_guest_list.key.network.eth[0-9].mac`: [required]: MAC address of virtual network interface (e.g. `52:54:00:16:01:bc`, etc )
* `virt_guest_list.key.network.eth[0-9].bridge`: [required]: The name of bridge interface in which vnet interface will be included (e.g. `br0`, `bridge1`, etc )
* `virt_guest_list.key.network.eth[0-9].model`: [optional, default `virtio`]: The qemu model of virtual network interface (e.g. `virtio`, `e1000`, `rtl8139` etc )
* `virt_guest_list.key.network.eth0.ip`: [optional]: Static IP address for main network interface
* `virt_guest_list.key.network.eth0.netmask`: [optional]: Netmask for main network interface
* `virt_guest_list.key.network.eth0.gateway`: [optional]: IP address of gateway for main network interface
* `virt_guest_list.key.network.eth0.dns`: [optional]:  Primary DNS server. This option supports only one DNS server.
* `virt_guest_list.key.vnc_enable`: [optional, default `false`]: Enable VNC server on qemu-kvm side to access a virtual machine

## Dependencies

serverbee.qemu_kvm role

#### Example(s)

##### Simple example

```yaml
---
- hosts: localhost 
  roles:
    - serverbee.virt_guest_manage
  vars:
    virt_guest_list:
      example-vm:
        uuid: 7eb09567-83b8-4aab-916e-24924d6a0f89
        disk:
          sda:
            vg: vg_local
            lv: lv_vm_example
            size: 10G
        network:
          eth0:
            mac: 52:54:00:16:01:bc
            bridge: br0
```

##### Full options example 

```yaml
---
- hosts: localhost 
  roles:
    - serverbee.virt_guest_manage
  vars:
    virt_network_list:
      br-nat0:
        router: 192.168.2.1
        netmask: 255.255.255.0
        start: 192.168.2.2
        end: 192.168.2.254

    virt_guest_list:
      example-vm:
        autostart: false
        uuid: 7eb09567-83b8-4aab-916e-24924d6a0f89
        cpu: 2 # Core
        ram: 4 # GiB
        disk:
          sda:
            type: block
            format: raw
            vg: vg_local
            lv: lv_vm_example-first
            size: 20%VG
            fstype: ext4
          vdb:
            type: file
            format: qcow2
            format_options: preallocation=metadata
            name: example-second-drive
            size: 5G
          sdc:
            device: /dev/nvme0n1
        network:
          eth0:
            mac: 52:54:00:16:01:bc
            bridge: br0
            model: virtio
            ip: 172.16.2.2
            netmask: 255.255.255.248
            gateway: 172.16.2.1
            dns: 172.16.2.1
          eth1:
            mac: 52:54:00:16:02:ba
            network: br-nat0
            model: e1000
        vnc_enable: true
```

#### What else?

##### Checking the progress of VM installation

First of all you can check if your VM responces to ping requests. If it doesn't responce then
you can use `virsh` cli tool to see the list of all your VMs and to check the progress of each installation.
Virsh command installs automatically as a dependency for this role. You can watch with using `console` option:

```bash
$ virsh list --all
$ virsh console example-vm
```

##### Timeout to finish the installation

There is a variable named as `virt_guest_kickstart_installation_timeout` and by default it set to 480 seconds.
If you have any problems with shutting down your VM before the finishing of your instalation then you have to increase the timeout.
Also it's better to set the nearest mirror at least in the same country with your hosting provider to increase a downloading speed.
You can do this with changing `virt_guest_mirror` and `virt_guest_os_location` variables.

##### Extra vars to manage only one VM

This option can be very useful if you have many virtual machines but want to manage only one of them.
It will run Ansible playbook faster and without applying some things for other virtual machines.
To do this you have to set an extra variable named `vm` when you apply your playbook:

```bash
$ ansible-playbook virt-guest-manage.yml --extra-vars vm=example-vm
```

It supports to pass only one name of virtual machine per one time.

##### Re-run an installation of VM

If you was playing with something before and want to re-run an installation again then first you have to remove all existing parts manually:

```bash
$ virsh list --all
$ virsh destroy example-vm
$ virsh undefine example-vm
$ lvremove vg_local/lv_vm_example
```

This role doesn't support a re-installing of VMs automatically to prevent any removing of existing and needed data.

#### License

GPLv3 license

#### Author Information

Vitaly Yakovenko
