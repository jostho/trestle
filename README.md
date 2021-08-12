# Trestle

![CI](https://github.com/jostho/trestle/workflows/CI/badge.svg)

This project has sources to prepare a [k3s](https://github.com/k3s-io/k3s) kubernetes cluster using [ansible](https://github.com/ansible/ansible).

## Environment

* fedora 34
* libvirt 7.0
* python 3.9
* ansible 2.9

## Setup VM

Create `Ubuntu 20.04 LTS` VMs on localhost using `virt-install`.
Use a [autoinstall](https://ubuntu.com/server/docs/install/autoinstall-reference) file in cloud-init user-data to automate the VM provisioning.
Setup password-less ssh login to all the VMs from the ansible control node. Update `hosts` file with the IP of the VMs.

Update `metallb.address_range` field in `group_vars/all.yml` file with the addresses for MetalLB load-balancer.

The VMs are created with the below configuration in my local setup
| Role | Count | CPU | Memory | Description |
| --- | --- | --- | --- | --- |
| client | 1 | 1 | 1000M | runs haproxy fronting the load-balancer, also provides kubectl/helm cli |
| master | 1 | 1 | 1600M | k3s server host |
| worker | 2 | 2 | 2000M | k3s agent hosts |

## Setup cluster

List hosts

    ansible-playbook --list-hosts site.yml

Run ansible playbooks to create the k3s cluster. Wait at least 60s between each playbook run.

    ansible-playbook -v -t common site.yml
    ansible-playbook -v -t master site.yml
    ansible-playbook -v -t worker site.yml
    ansible-playbook -v -t client site.yml

The cluster should be up and running in a few minutes.

## Client

Ssh into client host. Use `kubectl` to run commands against the cluster

    kubectl get nodes -o wide

List cluster addons

    kubectl get addons -A

List the helm charts installed

    helm ls -A
