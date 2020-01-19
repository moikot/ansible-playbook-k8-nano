# Ansible playbook for provisioning a Kubernetes cluster on Rock64 SBCs

An Ansible playbook that applies [Docker role](https://galaxy.ansible.com/geerlingguy/docker), [Kubernetes role](https://galaxy.ansible.com/geerlingguy/kubernetes), and [Kubernetes Dashboard role](https://galaxy.ansible.com/moikot/k8s_dashboard) to your nano-cluster.

**NOTE:** This Ansible playbook intended to be used for home experiments only.

## Requirements

One or more Rock64 SBCs with Linux installed and accessible via SSH.

## How to run

Clone the repository and change the inventory file (`inventory/k8s_nano.yml`).

```shell
git clone https://github.com/moikot/ansible-playbook-k8-nano.git
cd ansible-playbook-k8-nano
```

Run Ansible in a Docker container:

```shell
docker run -it --rm -v ${PWD}:/ansible pad92/ansible-alpine \
  ansible-playbook -i inventory k8s_nano.yml \
  --user=rock64 --ask-pass --ask-become-pass
```

or using a locally installed Ansible:

```shell
ansible-galaxy install -r requirements.yml
ansible-playbook -i inventory k8s_nano.yml --user=rock64 --ask-pass --ask-become-pass
```


## Inventory

Inventory file resides in `inventory` folder and contains important information about the provisioning. Here is an example of provisioning a three node cluster using Rock64 with IP address `192.168.88.247` as a master.

```yaml
all:
  children:
    masters:
      hosts:
        192.168.88.247:
          host_name: rock64-1
      vars:
        kubernetes_role: master

    nodes:
      hosts:
        192.168.88.248:
          host_name: rock64-2
        192.168.88.249:
          host_name: rock64-3
      vars:
        kubernetes_role: node
```

Enabling passwordless `sudo` for rock64 user.

```yaml
sudo_users:
  - name: rock64
    nopasswd: yes
```

The provided inventory specifies required Docker package architecture, suppresses `docker-compose` installation, and adds `rock64` to the `docker` group.

```yaml
docker_apt_arch: arm64
docker_install_compose: false

docker_users:
  - rock64
```

In the default configuration, the swap is disabled on all the instances using embedded role `pre_kuberntes`.

```yaml
disable_swap: yes
```

Kubernetes Dashboard is installed with the `Skip` login button enabled and full access to the cluster resources.

**IMPORTANT:** Please read [Access control](https://github.com/kubernetes/dashboard/wiki/Access-control) Wiki for more details.

```yaml
# Enable Skip login button and disable settings authorization.
k8s_dashboard_args:
  - --enable-skip-login
  - --disable-settings-authorizer
  - --auto-generate-certificates
  - --namespace=kubernetes-dashboard

# Allow Kubernetes dashboard to manage all the resources.
k8s_dashboard_cluster_role: cluster-admin
```
