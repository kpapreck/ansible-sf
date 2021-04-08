# Ansible NetApp SolidFire and VMware Automation
Project with two playbooks:
add_or_remove_esxi playbook will create NetApp-HCI like configuration on a generic vCenter or scaling an existing NetApp HCI
add_or_remove_solidfire takes the SolidFire portion of the above playbook for integrating SolidFire into an existing vCenter and/or adding additional SolidFire backed datastores into vCenter

# add_or_remove_esxi_host_to_vcenter_and_integrate_with_solidfire_storage.yml
Add or remove ESXi host to vCenter Ansible playbook
  Create vCenter Datacenter
  Create vCenter Cluster
  Add ESXi host to vCenter Cluster
    Setup all networking on ESXi host to match that of a NetApp HCI setup with the options of:
      2 or 6 cable VSS (only tested with 2-cable)
      2 or 6 cable VDS (only tested with 2-cable)
    Activate iSCSI initiator + setup port-binding to iSCSI ports
  Create SolidFire Account
  Create SolidFire Access Group
  Create SolidFire Volume
    Add SolidFire Volume to Access Group
    Add ESXi IQN to SolidFire Access Group
    Add SolidFire SVIP to ESXi host
  Create VMware datastore based on SolidFire volume
  Rescan VMware for new datastore to show up

# add_or_remove_solidfire_backed_datastores_and_setup_connections_to_esxi.yml
  Used as a standalone playbook for connecting SolidFire to an existing vCenter or to scale additional datastores if SolidFire is already connected
    Create SolidFire Account
    Create SolidFire Access Group
    Create SolidFire Volume
      Add SolidFire Volume to Access Group
      Add ESXi IQN to SolidFire Access Group
      Add SolidFire SVIP to ESXi host
    Create VMware datastore based on SolidFire volume
    Rescan VMware for new datastore to show up


## Requirements:
   - vCenter setup. Tested in 6.7 and 7.0
   - ESXi 6.7+ installed with management IP configured + root password set
   - elementOS >= 11.0
   - manual method:  
       - ansible >= 2.9.10
       - community_vmware 1.9.1-dev8 or newer
       - netapp element collection
       - modified vmware portgroup python code to allow for defined active uplinks
   -

## Required variables:
- copy secrets-example.yml to .secrets.yml and update all variables

## .secrets.yml variable descriptions

# solidfire variables
   #this is the management VIP connection to SolidFire
   elementsw_hostname: "<mvip>"

   elementsw_username: "admin"
   elementsw_password: "password"

   #this is your SVIP that you use for host connections to SolidFire
   send_target_address: "<solidfire svip>"

#vcenter variables for existing vCenter:
   vcenter_hostname: "<fqdn or ip>"
   vcenter_username: "administrator@vsphere.local"
   vcenter_password: "password"

#esxi_hostname to add to the cluster or to be used for adding/removing datastores
   esxi_hostname: "<esxi host fqdn or IP>"

#esxi_hostname credentials to add or remove host from vCenter (only required for add_or_remove_esxi_host workflow)
   esxi_username: "root"
   esxi_password: "root password of ESXi host"


## add_remove_vars.yml variable descriptions

################################################################################
# SolidFire specific variables (used for both playbooks)

#access group is the access group to use for the ESXi host provided. If the ESXi host is already added to vCenter and you are only running the add_or_remove_solidfire playbook you will want to use the same access group that you already have this host connected on.

# for example, if you want to scale ESXi and have it available to an exiting NetApp HCI cluster and all of that cluster's datastores, the default NetApp HCI access group is: NetApp-HCI.

# this will create a new access group if it does not exist. During "state=absent" if there are no remaining volumes in the access group it will delete the access group and any initiators.
  accessgroup: "NetApp-HCI-Ansible-Cluster"


# this will create a new SolidFire account if it does not exist. During "state=absent" if there are no remaining volumes in the account it will delete the account

  account: "{{ cluster_name }}"

#volname is the volume name/datastore name that will be used to add/remove a solidfire backed datastore
  volname: ansiblevolkp3

#optional QoS settings, otherwise will use 1000/15000/100000 and 2TB
  min: 500
  max: 15000
  burst: 100000
  volsize: 2048

################################################################################
# ALL REMAINING VARIABLES ARE NOT REQUIRED FOR add_or_remove_solidfire workflow
################################################################################

# vCenter variables for adding ESXi host

#datacenter_name = name of existing datacenter or name of new datacenter to create. state=absent will not remove the datacenter.

  datacenter_name: NetApp-HCI-Datacenter-01

#cluster_name = name of existing cluster or name of new cluster to create. state=absent will not remove the cluster. Do not use an existing cluster for this unless you first test it, as Ansible may change the settings of this existing cluster

  cluster_name: NetApp-HCI-Ansible-Cluster

# portgroup names and VLANs that are already configured on the switch to use for the new VDS switch that will be added to vCenter. Auotmation will add vMotion/iSCSI-A/iSCSI-B/VM_network to be similar to how NetApp HCI is configured with NDE. If you have a native VLAN, set it the id to 0:

#native VLAN example
  vMotion_vlan_id: 0

#vlan example:
  vMotion_portgroup_name: "vMotion"
  vMotion_vlan_id: 89

  VM_network_portgroup_name: "VM_Network"
  VM_network_vlan_id: 89

  iscsia_portgroup_name: "iSCSI-A"
  iscsib_portgroup_name: "iSCSI-B"
  iscsi_vlan_id: 91

# workflow supports both 2-cable and 6-cable setup with VSS or VDS.
  num_of_host_uplinks: 2 #supports either 2 or 6
  switch_type: "vds" # or "vss" are the only supported Variables

# esxi host information for the host you are adding to the cluster, which will require three additional IPs to be added for iSCSI-A, iSCSI-B, and vMotion:

  esxi_hostname: "winf-evo3-blade4.ntaplab.com"
  iscsia_ip: 10.45.91.217
  iscsib_ip: 10.45.91.218
  iscsi_gateway: 10.45.91.1
  iscsi_subnet: 255.255.255.0

  vmotion_ip: 10.45.89.191
  vmotion_gateway: 10.45.89.1
  vmotion_subnet: 255.255.255.0


  #VDS specific variables variables
      vds_name: "NetApp Ansible VDS"
      vds_management_portgroup_name: "Management Network"
      vds_management_vlan_id: 89

  #2-cable or 6-cable VSS variables for ESXi Host
      vswitch0_uplink1: vmnic0
      vswitch0_uplink2: vmnic1


  # 6-cable additional VSS variables for ESXi host

  # VM_Network and vMotion switch uplinks for vSwitch1
      #vswitch1_uplink1: vmnic4
      #vswitch1_uplink2: vmnic6

  # iSCSI switch uplinks for vSwitch2
      #vswitch2_uplink1: vmnic3
      #vswitch2_uplink2: vmnic5



## Basic setup
   - From host that has access to vCenter environment, use docker and grab the playbook files:
   docker run -it --rm -v $(pwd):/git alpine/git clone https://github.com/kpapreck/ansible-sf.git
   cd ansible-sf

   - Copy secrets-example.yml to .secret.yml
   - Edit variable files (as defined above)

   - Start container:
   ./run-container.sh

## Usage examples:
#Run add ESXi host workflow:
***ansible-playbook /scripts/add_or_remove_esxi_host_to_vcenter_and_integrate_with_solidfire_storage.yml -e "state=present"***

#Run add an additional datastore workflow:
to add an additional datastore, run the SolidFire specific workflow adding an extra var for a new volume name. you can also use the same workflow, but it will have all of the VMware verification steps first.
***ansible-playbook /scripts/add_or_remove_solidfire_backed_datastores_and_setup_connections_to_esxi.yml -e "state=present volname=addedvolname"***

#Remove added datastore (state=absent) using SolidFire workflow:
***ansible-playbook /scripts/add_or_remove_solidfire_backed_datastores_and_setup_connections_to_esxi.yml -e "state=absent volname=addedvolname"***

#Remove first datastore as defined in add_or_remove_vars.yml, cleaning up solidfire, reversing networking setup back to base ESXi and remove ESXi host from vCenter:
***ansible-playbook /scripts/add_or_remove_esxi_host_to_vcenter_and_integrate_with_solidfire_storage.yml -e "state=absent"***



## Usage examples:
Basic play with all vars in var files

***ansible-playbook AnsibleVolShare.yml***

Remove volume and other things from the play above

***ansible-playbook AnsibleVolShare.yml --extra-vars="state=absent"***

To skip using the NTFS and share hardening use the following

***ansible-playbook AnsibleVolShare.yml --skip-tags=harden***


##### Multiple varibles can be combined to execute a command vs updating each time the play is used.
***ansible-playbook AnsibleVolShare.yml --extra-vars="svm='svm01',volume_share='vol02',aggr='agggr1',vol_gb_size=50,netapp_hostname='Cluster02',netapp_username='Ansible-admin',netapp_password='Ansibe-password',useradmin_acct='AD-UserAdminGroup',usergroup_acct='ADUSERSGRP01',vol_qospolicy='extreme'"***


If you run into issues with ansible use the -vvv flag to show full output
The top issues in ansible are:
   - syntax errors spaces not tabs!
   - module installation or configuration of storage
