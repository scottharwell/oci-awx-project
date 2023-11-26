# Ansible Collection - harwell.oci

## Examples

### Deploy Site to Site VPN

```bash
ansible-navigator run harwell.oci.s2s-vpn_deploy.yml \
-i inventory/hosts \
--limit localhost \
--pae false --mode stdout --ce docker --ee true --pp always --eei quay.io/scottharwell/cloud-ee:latest \
--eev $HOME/.ssh:/home/runner/.ssh \
--eev $HOME/.oci:/home/runner/.oci \
--extra-vars "ansible_python_interpreter=/usr/bin/python3" \
--extra-vars "compartment_id=$OCI_COMPARTMENT_OCID"
--extra-vars "vpn_shared_secret=$VPN_SHARED_SECRET"
```
