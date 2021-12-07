# OCI Ansible Playbooks

This project contains playbooks that I use for testing, training, and other maintenance.

# Playbooks

This section will review the playbooks that require more explanation that the playbook files present on their own.

## Create Bastion Sessions

For servers that belong to a network that is not accessible from the public internet, we can use [OCI's Bastion service](https://docs.oracle.com/en-us/iaas/Content/Bastion/Concepts/bastionoverview.htm) to create jump hosts for Ansible without the need to setup a VM to act in this capacity.  This allows us to automate OCI resources without the Ansible Execution Environment having direct network access to the resource.

OCI uses the following diagram to demonstrate how the service works.

![Bastion Service Diagram](https://docs.oracle.com/en-us/iaas/Content/Bastion/images/bastion-overview-diagram.png)

The `create_bastion_session.yml` playbook automates establishing sessions through the bastion service in order to connect to those private virtual machines.

## OCI VM Setup