FROM quay.io/scottharwell/vscode-ansible:latest

ARG CONTAINER_USER="vscode"

# Ensure PIP is upgraded
RUN runuser -l ${CONTAINER_USER} -c "pip3 install pip --upgrade"

# Install OCI Collection
RUN runuser -l ${CONTAINER_USER} -c "pip3 install oci-cli"
RUN runuser -l ${CONTAINER_USER} -c "ansible-galaxy collection install oracle.oci"
RUN runuser -l ${CONTAINER_USER} -c "ansible-galaxy role install scottharwell.server_setup"