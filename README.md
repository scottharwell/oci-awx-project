# OCI Ansible Playbooks

This project contains playbooks that I use for testing, training, and other maintenance.

# Playbooks

This section will review the playbooks that require more explanation that the playbook files present on their own.

## Create Bastion Sessions

For servers that belong to a network that is not accessible from the public internet, we can use [OCI's Bastion service](https://docs.oracle.com/en-us/iaas/Content/Bastion/Concepts/bastionoverview.htm) to create jump hosts for Ansible without the need to setup a VM to act in this capacity.  This allows us to automate OCI resources without the Ansible Execution Environment having direct network access to the resource.

OCI uses the following diagram to demonstrate how the service works.

![Bastion Service Diagram](https://docs.oracle.com/en-us/iaas/Content/Bastion/images/bastion-overview-diagram.png)

The `create_bastion_session.yml` playbook automates establishing sessions through the bastion service in order to connect to those private virtual machines.  Once run, it will add each bastion session to the `all_hosts` and `bastion` host groups, which can then be used to automate against just as if they were exposed through the dynamic inventory.

## OCI VM Setup

This is a simple VM setup automation the configures servers with packages and CLI configs that I use commonly.  It has no OCI-specific use.

## RHEL Instances

I use these playbooks to demonstrate finding certain servers based on OCI tags and performing instance operations against those VMs.
