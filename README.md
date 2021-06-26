# Ansible playbook for provisioning a Kubernetes cluster on a cluster of Raspberry Pi

An Ansible playbook that applies [Docker role](https://galaxy.ansible.com/geerlingguy/docker) and [Kubernetes role](https://galaxy.ansible.com/geerlingguy/kubernetes) to a cluster of Raspberry Pi.

**NOTE:** This Ansible playbook intended to be used for home clusters only.

## Provisioning

You will need one or more Raspberry Pi single board computers connected to your home network.

Perform the following steps to provision a bare-metal Kubernetes cluster:
1. Flash Ubuntu Server 21.04 to SD cards, install them, and boot up.
2. Log in to your router and make sure the IP addresses assigned to them are static and bound to MAC addresses.
3. Log in to every Raspberry Pi via SSH and change the default password. You have to use the same password for all of them to run the playbook.
4. Clone https://github.com/moikot/ansible-playbook-k8-nano and make changes in the k8s_nano.yml inventory file specifying IP addresses of the Kubernetes master and nodes.
5. Navigate to the folder you cloned the playbook to and start the provisioning:
      a. Using Ansible in a Docker container:
      ```bash
      docker run -it --rm -v ${PWD}:/ansible pad92/ansible-alpine \
        ansible-playbook -i inventory k8s_nano.yml \
        --user=ubuntu --ask-pass --ask-become-pass
   ```
      b. Using a locally installed Ansible:
      ```bash
      ansible-galaxy install -r requirements.yml
      ansible-playbook -i inventory k8s_nano.yml \
        --user=ubuntu --ask-pass --ask-become-pass
   ```
 6. When Ansible finishes provisioning your cluster, add kubectl context using [metalogin](https://github.com/moikot/metalogin) utility:
```bash
ssh -o StrictHostKeyChecking=no -o LogLevel=ERROR ubuntu@[master IP address] \
  "sudo cat /etc/kubernetes/admin.conf" | \
  docker run -i --rm -v ~/.kube/:/kube moikot/metalogin -c /kube/config
```

7. Check that you have access to your cluster by running `kubectl get nodes` command.

## Inventory

The inventory file resides in `inventory` folder and contains essential information about the provisioning. Here is an example of provisioning a three-node cluster with IP address `192.168.88.247` assigned to the master node.

```yaml
all:
  children:
    masters:
      hosts:
        192.168.88.247:
          host_name: rpi4-1
      vars:
        kubernetes_role: master

    nodes:
      hosts:
        192.168.88.248:
          host_name: rpi4-2
        192.168.88.249:
          host_name: rpi4-3
      vars:
        kubernetes_role: node
```

Enabling passwordless `sudo` for the `ubuntu` user.

```yaml
sudo_users:
  - name: ubuntu
    nopasswd: yes
```

The provided inventory specifies the required Docker package architecture, suppresses `docker-compose` installation, and adds `ubuntu` user to the `docker` group.

```yaml
docker_apt_arch: arm64
docker_install_compose: false

docker_users:
  - ubuntu
```

In the default configuration, the swap is disabled on all the instances using embedded role `pre_kuberntes`.

```yaml
disable_swap: yes
```
