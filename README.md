## Virt guest manage role

This role use [official ansible virt module](https://docs.ansible.com/ansible/2.9/modules/virt_module.html) for managing virtual machines supported by libvirt.
So you can control all your vms directly via settings in group_vars or host_vars.
The main feature of this role is parallel installation of all virtual machines which set in `virt_guest_list`.

#### Variables

##### General

* `virt_guest_mirror`: [default: `http://mirror.eu.oneandone.net/linux/distributions`]: CentOS mirror what will be used to install OS
* `virt_guest_os_location`: [default: `{{ virt_guest_mirror }}/centos/7/os/x86_64`]: Location path where centos os componnets stored
* `virt_guest_kickstart_config_dir`: [default: `/tmp/kickstart`]: The path where kickstart files with be created
* `virt_guest_kickstart_config_url`: [default: `ip of a Hypervisor`]: The url for downloading kickstart configuration durring an installation
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
* `virt_guest_list.key.network`: [required]: Virt guest network(s) declarations
* `virt_guest_list.key.network.eth[0-9]`: [required]: The name of network interface in virtual machine (e.g. `eth0`, `eth1`, etc )
* `virt_guest_list.key.network.eth[0-9].mac`: [required]: MAC address of virtual network interface (e.g. `52:54:00:16:01:bc`, etc )
* `virt_guest_list.key.network.eth[0-9].bridge`: [required]: The name of bridge interface in which vnet interface will be included (e.g. `br0`, `bridge1`, etc )
* `virt_guest_list.key.network.eth[0-9].model`: [optional, default `virtio`]: The qemu model of virtual network interface (e.g. `virtio`, `e1000`, `rtl8139` etc )

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
          eth1:
            mac: 52:54:00:16:02:ba
            network: br-nat0
            model: e1000
```

##### Extra vars to manage only one VM

This option can be very useful if you have many virtual machines but want to manage only one of them.  
It will run Ansible playbook faster and without applying some things for other virtual machines.  
To do that you have to set an extra variable named `vm` when you apply your playbook:

```bash
$ ansible-playbook virt-guest-manage.yml --extra-vars vm=example-vm
```

It supports to pass only one name of virtual machine per one time.


#### License

GPLv3 license

#### Author Information

Vitaly Yakovenko
