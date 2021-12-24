# OCI Ansible Playbooks

This project contains playbooks that I use for testing, training, and other maintenance.

# Playbooks

This section will review the playbooks that require more explanation that the playbook files present on their own.

## Create Bastion Sessions

For servers that belong to a network that is not accessible from the public internet, we can use [OCI's Bastion service](https://docs.oracle.com/en-us/iaas/Content/Bastion/Concepts/bastionoverview.htm) to create jump hosts for Ansible without the need to setup a VM to act in this capacity.  This allows us to automate OCI resources without the Ansible Execution Environment having direct network access to the resource.

OCI uses the following diagram to demonstrate how the service works.

![Bastion Service Diagram](https://docs.oracle.com/en-us/iaas/Content/Bastion/images/bastion-overview-diagram.png)

The `create_bastion_session.yml` playbook automates establishing sessions through the bastion service in order to connect to those private virtual machines.  Once run, it will add each bastion session to the `all_hosts` and `bastion` host groups, which can then be used to automate against just as if they were exposed through the dynamic inventory.

### Running with Ansible Navigator

You can import the `create_bastion_sessions.yml` playbook into your own playbooks with `import_playbook`.  The `start_and_upgrade_all_vms.yml` file demonstrates chaining multiple playbooks together like a workflow.

Since the project follows the `ansible-runner` directory conventions, create a file at `./env/extravars` to manage the parameters that will be expected as part of the playbooks.  Create the file with the following parameters replacing with the values of your OCI configuration.

```yaml
---
bastion_name: "BASTION_OCID"
subnet_id: "OCID FOR THE SUBNET OF THE BASTION SESSION"
allowed_ips: ["CIDR OF APPROVED IPS TO CONNECT TO THE BASTION SERVICE"]
public_key: "{{ lookup('file', lookup('env','HOME') + '/.ssh/id_ed25519.pub') }}" # Replace with the public key that ansible will use to connect to the servers
target_user_name: opc # opc is the standard username for OCI VMs
target_resource_port: 22
# Add other params that are required for chained playbooks

```

Then, you can run `ansible-navigator` with an execution environment with OCI CLI and libraries to run the playbook.

```bash
ansible-navigator run project/create_bastion_sessions.yml \
-i inventory/inventory.oci.yml \
--pae false \
--extra-vars "@env/extravars" \
--mode stdout \
--eei quay.io/scottharwell/oci-execution-env:latest \
--eev $HOME/.oci:/home/runner/.oci \
--senv OCI_CONFIG_FILE="~/.oci/config" \
--senv OCI_PROFILE="REPLACE WITH YOUR OCI PROFILE...i.e. DEFAULT" \
--senv OCI_COMPARTMENT_ID="REPLACE WITH YOUR COMPARTMENT ID" \
--senv OCI_SSH_KEY_FILE="$HOME/.ssh/id_ed25519"
```

### Running with AWX or Ansible Controller

The playbook files lookup many of the OCI variables through environment variables.  You can create a custom credential type for OCI and use with these or other playbooks.  The following has been tested as a working configuration.

**Input Configuration**

```yaml
---
fields:
  - id: oci_region
    type: string
    label: Region
    secret: false
    help_text: OCI region used for API calls
    multiline: false
  - id: oci_profile_name
    type: string
    label: Profile Name
    secret: false
    help_text: Name for the profile for this credential
    multiline: false
  - id: oci_user_ocid
    type: string
    label: User OCID
    secret: false
    help_text: OCID for the user that this credential belongs to.
    multiline: false
  - id: oci_pub_fingerprint
    type: string
    label: Public key fingerprint
    secret: false
    help_text: The fingerprint for the public key associated to this credential.
    multiline: false
  - id: oci_tenancy_ocid
    type: string
    label: Tenancy OCID
    secret: false
    help_text: OCID for the tenancy that this credential belongs to.
    multiline: false
  - id: oci_compartment_ocid
    type: string
    label: Compartment OCID
    secret: false
    help_text: OCID for the compartment for this profile.
    multiline: false
  - id: oci_priv_key
    type: string
    label: Private Key
    format: ssh_private_key
    secret: true
    help_text: OCI private key
    multiline: true
  - id: oci_ssh_key
    type: string
    label: Private SSH Key
    format: ssh_private_key
    secret: true
    help_text: SSH private key for OCI hosts and Bastion service
    multiline: true
required:
  - oci_region
  - oci_profile_name
  - oci_user_ocid
  - oci_pub_fingerprint
  - oci_tenancy_ocid
  - oci_priv_key
  - oci_ssh_key

```

**Injector Configuration**

```yaml
---
env:
  OCI_PROFILE: '{{ oci_profile_name }}'
  OCI_CONFIG_FILE: '{{ tower.filename.oci_config }}'
  OCI_SSH_KEY_FILE: '{{ tower.filename.oci_ssh_key }}'
  OCI_USER_KEY_FILE: '{{ tower.filename.oci_key }}'
  OCI_COMPARTMENT_ID: '{{ oci_compartment_ocid }}'
file:
  template.oci_key: '{{ oci_priv_key }}'
  template.oci_config: |-
    [{{ oci_profile_name }}]
    user={{ oci_user_ocid }} 
    fingerprint={{ oci_pub_fingerprint }} 
    key_file={{ tower.filename.oci_key }} 
    tenancy={{ oci_tenancy_ocid }} 
    region={{ oci_region }}
  template.oci_ssh_key: '{{ oci_ssh_key }}'

```

Once you have created this custom credential type, then create a credential with the values from this type filled in.  Lastly, use this credential with the AWX / Automation Controller templates that you create.

## OCI VM Setup

This is a simple VM setup automation the configures servers with packages and CLI configs that I use commonly.  It has no OCI-specific use but demonstrates creating a workflow of chained playbooks to start servers, create bastion sessions with private servers, update them all, and then shut down the servers that do not run all of the time.

## RHEL Instances

I use these playbooks to demonstrate finding certain servers based on OCI tags and performing instance operations against those VMs.
