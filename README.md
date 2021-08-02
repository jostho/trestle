# Trestle

This project has sources to prepare a [k3s](https://github.com/k3s-io/k3s) cluster using [ansible](https://github.com/ansible/ansible).

## Environment

* fedora 34
* libvirt 7.0
* python 3.9
* ansible 2.9

## Setup

Create `Ubuntu 20.04 LTS` VMs on local using `virt-install`. Update `hosts` file with the IPs of the VMs.

Create a `~/.ansible.cfg` config file, with the below contents

    [defaults]
    host_key_checking = False
    retry_files_enabled = False
    callback_whitelist = profile_tasks
    [ssh_connection]
    pipelining = True

Check ansible connectivity to the hosts

    ansible-playbook -i hosts --list-hosts site.yml

## Cluster

Run ansible playbooks to create the k3s cluster

    ansible-playbook -v -i hosts -t common site.yml
    ansible-playbook -v -i hosts -t master site.yml
    ansible-playbook -v -i hosts -t worker site.yml
    ansible-playbook -v -i hosts -t client site.yml

## Client

Ssh into client host and get the kubeconfig file

    scp <master_ip>:/etc/rancher/k3s/k3s.yaml ~/.kube

Change the `server` line in kubeconfig, to point to master

    sed -i 's/127.0.0.1/<master_ip>/g' ~/.kube/k3s.yaml

Use `kubectl` to run commands against the cluster

    kubectl get nodes
