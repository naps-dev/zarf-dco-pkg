# Zarf DCO Package

This is a simple Defensive Cyber Operations (CDO) package created with Zarf.

## Quick Start

*Note: This assumes the destination Kubernetes cluster is configured with a local path provisioner and accessible*

1. Build and deploy the package
    ```bash
    zarf p c --confirm --skip-sbom
    zarf init --storage-class "local-path"
    zarf p d
    ```

## Bottom Up

1. x86-64 Linux workstation
    * Zarf installed
1. Docker login to registry 1
    ```bash
    # Run on workstation
    zarf tools registry login -u [username] --password-stdin registry1.dso.mil
    ```
1. Build the DCO package
    ```bash
    # Run on workstation
    zarf package create --confirm --skip-sbom
    ```
1. Homelab Server or cluster of devices
    * At least 64 GB RAM
    * Recommend 500 GB storage for local testing
1. Rocky 8.6 Operating System
1. Disable firewalld
    ```bash
    # Run on destination server
    sudo systemctl disable firewalld --now
    ```
1. Do a couple of other nice things
    ```bash
    # Run on destination server
    sudo sysctl -w vm.max_map_count=1524288
    sudo sysctl -w fs.file-max=1000000
    ulimit -n 1000000
    ulimit -u 8192
    sudo sysctl --load
    sudo swapoff -a
    sudo sysctl fs.inotify.max_user_instances=8192
    sudo sysctl -p
    ```
1. Install kubectl
    ```bash
    # Run on destination server
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    ```
1. Write the RKE2 config file
    ```bash
    # Run on destination server
    sudo mkdir -p /etc/rancher/rke2
  
    sudo bash -c 'cat > /etc/rancher/rke2/config.yaml' << EOF
    bind-address: "192.168.1.73"
    write-kubeconfig-mode: "0644"
    disable: 
      - "rke2-ingress-nginx"
    EOF
    ```
1. Install and start RKE2
    ```bash
    # Run on destination server
    sudo su
    curl -sfL https://get.rke2.io | sh -
    systemctl enable rke2-server.service
    systemctl start rke2-server.service
    ```
1. Grab the config file
    ```bash
    # Run on workstation
    scp [user]@[server]:/etc/rancher/rke2/rke2.yaml ~/.kube/config
    ```
1. Install local path provisioner
   ```bash
   # Run on workstation
   kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.23/deploy/local-path-storage.yaml
   ```
1. Zarf init
    ```bash
    # Run on workstation
    zarf init --storage-class "local-path"
    ```
1. Install this DCO Package
    ```bash
    # Run on workstation
    zarf package deploy 
    ```