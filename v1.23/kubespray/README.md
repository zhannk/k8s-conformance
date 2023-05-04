Download and install Kubespray 2.19.0
```shell
git clone https://github.com/kubernetes-sigs/kubespray.git
git checkout v2.19.0
```

Create vagrant configuration

```shell
mkdir -p vagrant
cat > ./vagrant/config.rb <<__EOF__
$vm_cpus = 4
$vm_memory = 4096
$num_instances = 3
$os = "ubuntu2004"
$subnet = "10.2.20"
$inventory = "inventory/k8s-conformance"
$network_plugin = "calico"
__EOF__
mkdir -p inventory/k8s-conformance
```

Start vagrant deployment and access kube-master
```shell
$ vagrant up
$ vagrant ssh k8s-1
```

Run [Sonobuoy](https://github.com/heptio/sonobuoy) as instructed [here](https://github.com/cncf/k8s-conformance/blob/master/instructions.md).
