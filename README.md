Infrastructure VMWare role
---------
This role reads the virtual machine definition from the host variables of an instance and sets up the virtual machine in VMWare accordingly.

General rules for this role:

Supported actions:
* Create new virtual machines
* Modify virtual machines: add memory, vCPU's, disks and network interfaces when the virtual machine is running
* Modify virtual machines: remove memory and vCPU's when the virtual machine is powered off
* Modify vitual machines: memory & CPU without having to reboot (on the fly) has been enable "hotadd" option

Not supported actions:
* Remove disks from virtual machine
* Remove network interfaces from the virtual machine
* Remove complete virtual machine
* Changing the hostname and/or primary interface of the virtual machine


__Requirements__
This role cannot be called without a defined `instances` variable. If the variable `instances` is defined then the VM definition must be present in the host_variables. Further more, the VMWare credentials need to be attached to this play, otherwise it cannot connect to the specified vCenter server.

The following variable is required in the inventory to make this role work
- infrastructure_template_disk_size: (The template is build with 35gb so the default value is 35)

__Usage__
In order for this role to start making changes, the following items are required:

an `instances` variable (list), filled with hostnames in FQDN, in example:
* node1.domain
* node2.domain

A VMWare definition in the host variables, in example (the host variables will consist of the combination of host specific variables and the appropriate group variables, the example below is the resulting set of all VMWare related host variables as present in Ansible):
```
{
 "dns_servers": [
  "192.168.140.2",
  "192.168.90.2",
  "192.168.70.2"
 ],
 "dns_domain": "domain",
 "catalog": {
  "s": {
   "vm_vcpus": "2",
   "vm_memory_mb": "8192"
  },
  "m": {
   "vm_vcpus": "4",
   "vm_memory_mb": "32768"
  },
  "l": {
   "vm_vcpus": "8",
   "vm_memory_mb": "65536"
  }
 },
 "meta": {
  "platform": {
   "version": "1.0",
   "name": "ICT platform"
  }
 },
 "primary_interface": {
  "ip": "10.95.6.4",
  "network": "frontend1"
 },
 "vmware": {
  "datacenter": "DC_UA_MER3",
  "datastore_filter": "UA_MER3_PROD",
  "cluster": "CLU_UA_MER3",
  "vm_folder": "Linux/PaaS-ICT/test",
  "networks": {
   "backend1": {
    "vmware_name": "CIC_Int_GR (vlan430)",
    "netmask": "255.255.255.0",
    "network": "10.95.7.0"
   },
   "frontend1": {
    "vmware_name": "PROD_GR (vlan767)",
    "netmask": "255.255.255.0",
    "gateway": "10.95.6.1",
    "network": "10.95.6.0"
   }
  },
  "source_template": "rhel74_template"
 },
 "size": "s",
 "location_name": "DEV",
 "hostname": "node4.domain",
 "type": "vm",
 "vcenter_server": "<vcenter>.domain"
}

```
The definition snippet above will result in:
* A single virtual machine with 2 vCPU's and 8GB of memory in cluster CLU_UA_MER3 in datacenter DC_UA_MER3 as defined on vCenter server adcmvca04
* A single network interface attached to the system, with IP and DNS configuration, mapped to network PROD_GR (vlan7670) in VMWare
* A single 60GB boot disk (default setting)
* The virtual machine is cloned from template rhel74_template
* The virtual machine will be placed in folder Linux/PaaS-ICT/test
###### Mandatory keys
As stated, the definition above is a combination of a host specific definition and group definitions. The minimal host specific definition for the VMWare playbook to create a virtual machine is:
```
"meta": {
  "platform": {
     "name": "ICT platform besturing", --> Platform name
     "version": "1.0", --> Platform version definition
     "contact": "<email>" --> Contact person for this platform
  }
}

{
  "type": "vm", --> Must be provided, otherwise the pipeline will not add this instance to the instances list used by this playbook
  "hostname": "node4.domain", --> The hostname for the new/modified system, in FQDN
  "primary_interface": {
    "network": "frontend1", --> The VMWare network to use to attach the network interface to (provided by group variables)
    "ip": "10.95.6.4" --> The IP address to be used for the primary interface
  },
  "size": "s" --> The size of the instance (defined as catalog item)
}

```
###### Optional keys
Additional disks (list of dictionaries):
```
"disks": [
  {
    "size": 20, --> size of the disk in GB
    "vg": "u01" --> Volume group to which this disk as a whole will be assigned
  },
  {
    "size": 20,
    "vg": "orafra"
  }
]
```
Root disk size (integer):
```
"root_disk_size": 100 --> Size of the root disk in GB, must be larger than 60
```
Additional network interfaces (list of dictionaries):
```
"additional_interfaces": [
  {
    "network": "backend1", --> The VMWare network to use to attach this network interface to (provided by group variables)
    "ip": "10.95.7.27" --> The IP address to be used on the interface
  }
]
```

__Role Variables__
This role uses the following default variables:

```
"catalog": {
  "s": {
   "vm_vcpus": "2",
   "vm_memory_mb": "8192"
  },
  "m": {
   "vm_vcpus": "4",
   "vm_memory_mb": "32768"
  },
  "l": {
   "vm_vcpus": "8",
   "vm_memory_mb": "65536"
  }

```
These are set in the infrastructure buildingblock, so they are no longer needed in the inventory.

__Dependencies__
This playbook is dependant on the following Ansible Galaxy roles:

- community.vmware

__License__
tbd

__Author Information__

