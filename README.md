# Trestle

![CI](https://github.com/jostho/trestle/actions/workflows/ci.yml/badge.svg)

This project has sources to prepare a [k3s](https://github.com/k3s-io/k3s) kubernetes cluster using [ansible](https://github.com/ansible/ansible).

## Environment

* fedora 41
* libvirt 10.6
* python 3.13
* ansible 2.16

## Setup VM

Create `Ubuntu 24.04 LTS` VMs on localhost using `virt-install`.
Use a [autoinstall](https://ubuntu.com/server/docs/install/autoinstall-reference) file in cloud-init user-data to automate the VM provisioning.
Setup password-less ssh login to all the VMs from the ansible control node. Update [hosts](hosts) file with the IP of the VMs.

Update `metallb.address_range` and `metallb.lb_ip` fields in [group_vars/all.yml](group_vars/all.yml) file with the addresses for MetalLB load-balancer.

The VMs are created with the below configuration in my local setup
| Role | Count | CPU | Memory | Disk | Description |
| --- | --- | --- | --- | --- | --- |
| client | 1 | 1 | 1200M | 12G | provides cli tools |
| master | 1 | 1 | 2400M | 12G | k3s server host |
| worker | 2 | 2 | 2400M | 16G | k3s agent hosts |

## Setup cluster

List hosts

    ansible-playbook --list-hosts site.yml

Run ansible playbooks to create the k3s cluster. Wait at least 60s between each playbook run.

    ansible-playbook -v -t common site.yml
    ansible-playbook -v -t master site.yml
    ansible-playbook -v -t worker site.yml
    ansible-playbook -v -t client site.yml

The cluster should be up and running in a few minutes.

The below helm charts are installed on the k3s cluster as addons
1. [traefik](https://github.com/traefik/traefik-helm-chart) (ingress)
1. [kube-prometheus-stack](https://prometheus-community.github.io/helm-charts) (monitoring)
1. [loki](https://grafana.github.io/helm-charts) (log aggregation)
1. [argocd](https://github.com/argoproj/argo-helm) (continuous delivery)
1. [kubernetes-dashboard](https://github.com/kubernetes/dashboard) (cluster dashboard)
1. [metallb](https://github.com/metallb/metallb) (load-balancer)
1. [nfs-subdir-external-provisioner](https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner) (nfs client provisioner)

## Client

Ssh into `client` host. Use `kubectl` to run commands against the cluster

    kubectl get nodes -o wide

List cluster addons

    kubectl get addons -A

List the helm charts installed

    helm ls -A

The `client` host also runs the below components
1. NFS server which is used by the nfs client provisioner - to host persistent volumes
1. Haproxy which fronts the traefik ingress
