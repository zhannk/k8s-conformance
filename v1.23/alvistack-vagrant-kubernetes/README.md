# AlviStack - Vagrant Box Packaging for Kubernetes

For running k8s conformance test we need 2 vagrant instances as master
and 1 vagrant instance as node with following minimal system
requirement, e.g.

  - host
      - libvirt
      - nested virtualization enabled
      - Ubuntu 22.04
      - 8 CPUs
      - 32GB RAM
  - `kube01`
      - kubernetes master, etcd
      - cri-o, flannel
      - Ubuntu 22.04
      - IP: 192.168.121.101/24
      - 2 CPUs
      - 8GB RAM
  - `kube02`
      - kubernetes master, etcd
      - cri-o, flannel
      - Ubuntu 22.04
      - IP: 192.168.121.102/24
      - 2 CPUs
      - 8GB RAM
  - `kube03`
      - kubernetes node, etcd
      - cri-o, flannel
      - Ubuntu 22.04
      - IP: 192.168.121.103/24
      - 2 CPUs
      - 8GB RAM

## Bootstrap Host

Install some basic pacakges for host:

    apt update
    apt full-upgrade
    apt install aptitude git linux-generic-hwe-22.04 openssh-server python3 rsync vim

Install Libvirt:

    apt-get update
    apt-get install -y binutils bridge-utils dnsmasq-base ebtables gcc libarchive-tools libguestfs-tools libvirt-clients libvirt-daemon-system libvirt-dev make qemu-system qemu-utils ruby-dev virt-manager

Install Vagrant:

    echo "deb [arch=amd64] https://apt.releases.hashicorp.com jammy main" | tee /etc/apt/sources.list.d/hashicorp.list
    curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor | tee /etc/apt/trusted.gpg.d/hashicorp.gpg > /dev/null
    apt-get update
    apt-get install -y vagrant
    vagrant plugin install vagrant-libvirt

## Bootstrap Ansible

Install Ansible (see
<https://software.opensuse.org/download/package?package=ansible&project=home%3Aalvistack>):

    echo "deb http://download.opensuse.org/repositories/home:/alvistack/xUbuntu_22.04/ /" | tee /etc/apt/sources.list.d/home:alvistack.list
    curl -fsSL https://download.opensuse.org/repositories/home:alvistack/xUbuntu_22.04/Release.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/home_alvistack.gpg > /dev/null
    apt-get update
    apt-get install -y ansible python3-ansible-lint python3-docker python3-netaddr python3-vagrant

Install Molecule:

    echo "deb http://download.opensuse.org/repositories/home:/alvistack/xUbuntu_22.04/ /" | tee /etc/apt/sources.list.d/home:alvistack.list
    curl -fsSL https://download.opensuse.org/repositories/home:alvistack/xUbuntu_22.04/Release.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/home_alvistack.gpg > /dev/null
    apt-get update
    apt-get install -y python3-molecule python3-molecule-docker python3-molecule-vagrant

GIT clone Vagrant Box Packaging for Kubernetes
(<https://github.com/alvistack/vagrant-kubernetes>):

    mkdir -p /opt/vagrant-kubernetes
    cd /opt/vagrant-kubernetes
    git init
    git remote add upstream https://github.com/alvistack/vagrant-kubernetes.git
    git fetch --all --prune
    git checkout upstream/develop -- .
    git submodule sync --recursive
    git submodule update --init --recursive

## Deploy Kubernetes

Deploy kubernetes:

    cd /opt/ansible-collection-kubernetes
    export _MOLECULE_INSTANCE_NAME="$(pwgen -1AB 12)"
    molecule converge -s kubernetes-1.23-libvirt -- -e 'kube_release=1.23'
    molecule verify -s kubernetes-1.23-libvirt

All instances could be SSH and switch as root with `sudo su -`, e.g.

    cd /opt/ansible-collection-kubernetes
    molecule login -s kubernetes-1.23-libvirt -h $_MOLECULE_INSTANCE_NAME-1

Check result:

    root@kube01:~# kubectl get node
    NAME     STATUS   ROLES                  AGE     VERSION
    kube01   Ready    control-plane,master   9m38s   v1.23.10
    kube02   Ready    control-plane,master   8m32s   v1.23.10
    kube03   Ready    <none>                 8m14s   v1.23.10
 
    root@kube01:~# kubectl get pod --all-namespaces
    NAMESPACE      NAME                             READY   STATUS    RESTARTS   AGE
    kube-flannel   kube-flannel-ds-5z9q7            1/1     Running   0          70s
    kube-flannel   kube-flannel-ds-7bf94            1/1     Running   0          70s
    kube-flannel   kube-flannel-ds-99tj8            1/1     Running   0          70s
    kube-system    coredns-bd6b6df9f-cqzkx          1/1     Running   0          70s
    kube-system    coredns-bd6b6df9f-tcv7m          1/1     Running   0          70s
    kube-system    kube-addon-manager-kube01        1/1     Running   2          80s
    kube-system    kube-addon-manager-kube02        1/1     Running   2          80s
    kube-system    kube-apiserver-kube01            1/1     Running   3          80s
    kube-system    kube-apiserver-kube02            1/1     Running   3          80s
    kube-system    kube-controller-manager-kube01   1/1     Running   3          80s
    kube-system    kube-controller-manager-kube02   1/1     Running   4          80s
    kube-system    kube-proxy-brwdv                 1/1     Running   0          70s
    kube-system    kube-proxy-ppn7b                 1/1     Running   0          70s
    kube-system    kube-proxy-qn7vg                 1/1     Running   0          70s
    kube-system    kube-scheduler-kube01            1/1     Running   3          80s
    kube-system    kube-scheduler-kube02            1/1     Running   4          80s

## Run Sonobuoy

Run sonobuoy for conformance test as official procedure
(<https://github.com/cncf/k8s-conformance/blob/master/instructions.md>):

    root@kube01:~# sonobuoy run --mode=certified-conformance
    
    root@kube01:~# sonobuoy status
    
    root@kube01:~# sonobuoy retrieve
