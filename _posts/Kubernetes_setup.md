# Local-machine Solutions

## Before you begin

VT-x or AMD-v virtualization must be enabled in your computer’s BIOS.

## Install a Hypervisor

>If you do not already have a hypervisor installed, install one now.
    For OS X, install xhyve driver, VirtualBox, or VMware Fusion, or HyperKit.
    For Linux, install VirtualBox or KVM.
    For Windows, install VirtualBox or Hyper-V.

My environment is Linux, so I download the file from the official website.
Then run the command :
`sudo dpkg -i virtualbox-5.2_5.2.4-119785~Ubuntu~xenial_amd64.deb`

## Install kubectl

1. Download the latest release with the command:
run the command
`curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl`


2. Make the kubectl binary executable.

` chmod +x ./kubectl`

3. Move the binary in to your PATH.

 `sudo mv ./kubectl /usr/local/bin/kubectl`

## Install Minikube
 `curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.24.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/`

 
