# Trestle

![CI](https://github.com/jostho/trestle/workflows/CI/badge.svg)

This project has sources to prepare a [k3s](https://github.com/k3s-io/k3s) kubernetes cluster using [ansible](https://github.com/ansible/ansible).

## Environment

* fedora 34
* libvirt 7.0
* python 3.9
* ansible 2.9

## VM

Create `Ubuntu 20.04 LTS` VMs on localhost using `virt-install`.
Provide a [autoinstall](https://ubuntu.com/server/docs/install/autoinstall-reference) file in cloud-init user-data to automate the VM provisioning.
Setup password-less ssh login to all the VMs from the ansible control node. Update `hosts` file with the IP of the VMs.

## Cluster

List hosts

    ansible-playbook --list-hosts site.yml

Run ansible playbooks to create the k3s cluster

    ansible-playbook -v -t common site.yml
    ansible-playbook -v -t master site.yml
    ansible-playbook -v -t worker site.yml
    ansible-playbook -v -t client site.yml

## Client

Ssh into client host. Use `kubectl` to run commands against the cluster

    kubectl get nodes

Taint the master node

    kubectl taint nodes <master> node-role.kubernetes.io/master=:NoSchedule

Label the worker nodes

    kubectl label nodes <worker> node-role.kubernetes.io/worker=true role=worker
