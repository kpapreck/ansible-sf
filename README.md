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

# solidfire variables
   elementsw_hostname: "<mvip>"
   elementsw_username: "admin"
   elementsw_password: "password"
   send_target_address: "<solidfire svip>"

#vcenter variables:
   vcenter_hostname: "<fqdn or ip>"
   vcenter_username: "administrator@vsphere.local"
   vcenter_password: "password"

#esxi_hostname to add to the cluster or to be used for adding/removing datastores
   esxi_hostname: "<esxi host fqdn or IP>"

#esxi_hostname credentials to add or remove host from vCenter
   #esxi_username and esxi_password are only required for add_or_remove_esxi_host workflow
   esxi_username: "root"
   esxi_password: "root password of ESXi host"



   - On NetApp Cluster:
     - Create a dedicated admin account ssh,ontapi,console
     - ***set -priv advanced; system services web modify -http-enabled true*** command set on storage

## .secrets.yml variable descriptons
    -
## Basic setup
   - Install prerequisites
   - Configure storage for ansible
   - Clone repo
   - Review the AnsibleVolShare.yml
   - Update secret.yml, uservars.yml, defaultvars.yml


## Usage examples:
Basic play with all vars in var files

***ansible-playbook AnsibleVolShare.yml***

Remove volume and other things from the play above

***ansible-playbook AnsibleVolShare.yml --extra-vars="state=absent"***

To skip using the NTFS and share hardening use the following

***ansible-playbook AnsibleVolShare.yml --skip-tags=harden***


##### Multiple varibles can be combined to execute a command vs updating each time the play is used.
***ansible-playbook AnsibleVolShare.yml --extra-vars="svm='svm01',volume_share='vol02',aggr='agggr1',vol_gb_size=50,netapp_hostname='Cluster02',netapp_username='Ansible-admin',netapp_password='Ansibe-password',useradmin_acct='AD-UserAdminGroup',usergroup_acct='ADUSERSGRP01',vol_qospolicy='extreme'"***


Note: the NVE encryption flag has been set to true by default. set encrypt="false" if this is not the desired behavior.

If you run into issues with ansible use the -vvv flag to show full output
The top issues in ansible are:
   - syntax errors spaces not tabs!
   - module installation or configuration of storage
   - version compatibility issues older versions of ontap work with some of the features not all.
