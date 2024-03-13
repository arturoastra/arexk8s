Arex Master Node
=================

Initialize the Kubernetes cluster with Kubeadm
----------------------------------------------

We want to initialize our cluster with the pod network CIDR specifically set to `192.168.0.0/16` as this is the default range utilized by the Calico network plugin. If needed, it is possible to set a different [RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918) range during `kubeadm init` and configure Calico to use that range. Instructions for configuring Calico for a different IP range is noted below in [Pod Network](#pod-network).



    kubeadm init --pod-network-cidr=192.168.0.0/16
    

KubeConfig
----------

If you want to permanently enable `kubectl` access for the `root` account, you will need to  the Kubernetes admin configuration to your home directory as shown below.



    mkdir -p $HOME/.kube && \
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && \
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    

To instead set `KUBECONFIG` for a single session simply run:



    export KUBECONFIG=/etc/kubernetes/admin.conf
    

Allowing pods to run on the Control Plane
-----------------------------------------

  **Note:** This step is optional for multi-node installations of Kubernetes and required for single-node installations.

If you are running a single-node SLATE cluster, you’ll want to remove the `NoSchedule` taint from the Kubernetes Control Plane. This will allow general workloads to run along-side of the Kubernetes Control Plane processes. In larger clusters, it may instead be desirable to prevent “user” workloads from running on the Control Plane, especially on very busy clusters where the K8s API is servicing a large number of requests. If you are running a large, multi-node cluster then you may want to skip this step.

To remove the Control Plane taint:



    kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule-
    

You might want to adjust the above command based on the role your Control Plane node holds. You can find this out by running:



    kubectl get nodes
    

This should tell you the role(s) your Control Plane node holds. You can also adjust the command to remove the taint accordingly.

Pod Network
-----------

In order to enable Pods to communicate with the rest of the cluster, you will need to install a networking plugin. There are a large number of possible networking plugins for Kubernetes. SLATE clusters generally use Calico, although other options should work as well.

To install Calico, you will simply need to apply the appropriate Kubernetes manifests beginning with the operator:



    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/tigera-operator.yaml
    

If you haven’t changed the default IP range then create the boilerplate custom resources manifest:



    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/custom-resources.yaml
    

If you have changed the IP range to anything other than `192.168.0.0/16` in the `kubeadm init` command above, you will need to first download the boilerplate [custom-resources.yaml](https://github.com/projectcalico/calico/blob/master/manifests/custom-resources.yaml) file from Project Calico on GitHub, then update its IP range under `spec/calicoNetwork/ipPools/blockSize` and `CIDR`. Finally, create the custom resources manifest:



    kubectl create -f /path/to/custom-resources.yaml
    

After approximately five minutes, your master node should be ready. You can check with `kubectl get nodes`:



    [root@your-node ~]# kubectl get nodes
    NAME                           STATUS   ROLES                  AGE     VERSION
    your-node.your-domain.edu      Ready    control-plane        2m50s   v1.28.0
    

Load Balancer/MetalLB
---------------------

When using the Kubernetes service type of Load Balancer, [MetalLB](https://metallb.org/) will attach an IP address to the service from a pool of address ranges you specify in the MetalLB configuration. The SLATE team usually uses MetalLB to automatically attach public IP addresses to a service, but it can be used with RFC1918 IP addresses as well.

Installing and configuring MetalLB is done in three steps: installation, configuration, and advertisement. (There is a fourth step if you are using virtual machines managed by OpenStack.)

### Step 1: Installation

Run this command to install MetalLB:



    METALLB_VERSION=0.13.12 && \
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v${METALLB_VERSION}/config/manifests/metallb-native.yaml
    

### Step 2: Configuration

The following command creates an IP Address Pool file. Replace the addresses in this IPAddressPool configuration with the available IP addresses for your cluster. As noted in the example, you can specify a range with a CIDR or a hyphen. You may configure more than one range of IP addresses. The example configures two ranges (make sure to remove the second range if you only need one).



    cat <<EOF > /tmp/metallb-ipaddrpool.yml
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: first-pool
      namespace: metallb-system
    spec:
      addresses:
      - 192.168.10.0/24
      - 192.168.9.1-192.168.9.5
    EOF
    

You can now apply the configuration file to Kubernetes:



    kubectl create -f /tmp/metallb-ipaddrpool.yml
    

### Step 3: Advertisement

Once configured, we need to tell MetalLB how to advertise the IP addresses in the pool. In this case we are configuring the pool to use Layer 2 advertisement. [MetalLB also supports BGP](https://metallb.org/configuration/_advanced_bgp_configuration/).



    cat <<EOF > /tmp/metallb-ipaddrpool-advert.yml
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
      name: example
      namespace: metallb-system
    spec:
      ipAddressPools:
      - first-pool
    EOF
    

Apply the advertisement file to Kubernetes:



    kubectl create -f /tmp/metallb-ipaddrpool-advert.yml
    

### Step 4 (if you are running VMs on OpenStack): MetalLB on OpenStack

The [MetalLB documentation](https://metallb.universe.tf/faq/#is-metallb-working-on-openstack) notes the following for OpenStack managed virtual machines:

You can run a Kubernetes cluster on OpenStack VMs, and use MetalLB as the load balancer. However, you have to disable OpenStack’s ARP spoofing protection if you want to use L2 mode. You must disable it on all the VMs that are running Kubernetes.

By design, MetalLB’s L2 mode looks like an ARP spoofing attempt to OpenStack, because we’re announcing IP addresses that OpenStack doesn’t know about. There’s currently no way to make OpenStack cooperate with MetalLB here, so we have to turn off the spoofing protection entirely.

[Next Page »](/docs/cluster/manual/slate-worker-node.html)
Initialize the Kubernetes cluster with Kubeadm
----------------------------------------------

We want to initialize our cluster with the pod network CIDR specifically set to `192.168.0.0/16` as this is the default range utilized by the Calico network plugin. If needed, it is possible to set a different [RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918) range during `kubeadm init` and configure Calico to use that range. Instructions for configuring Calico for a different IP range is noted below in [Pod Network](#pod-network).


    kubeadm init --pod-network-cidr=192.168.0.0/16
    

KubeConfig
----------

If you want to permanently enable `kubectl` access for the `root` account, you will need to copy the Kubernetes admin configuration to your home directory as shown below.


    mkdir -p $HOME/.kube && \
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && \
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    

To instead set `KUBECONFIG` for a single session simply run:

Copy

    export KUBECONFIG=/etc/kubernetes/admin.conf
    

Allowing pods to run on the Control Plane
-----------------------------------------

  **Note:** This step is optional for multi-node installations of Kubernetes and required for single-node installations.

If you are running a single-node SLATE cluster, you’ll want to remove the `NoSchedule` taint from the Kubernetes Control Plane. This will allow general workloads to run along-side of the Kubernetes Control Plane processes. In larger clusters, it may instead be desirable to prevent “user” workloads from running on the Control Plane, especially on very busy clusters where the K8s API is servicing a large number of requests. If you are running a large, multi-node cluster then you may want to skip this step.

To remove the Control Plane taint:


    kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule-
    

You might want to adjust the above command based on the role your Control Plane node holds. You can find this out by running:


    kubectl get nodes
    

This should tell you the role(s) your Control Plane node holds. You can also adjust the command to remove the taint accordingly.

Pod Network
-----------

In order to enable Pods to communicate with the rest of the cluster, you will need to install a networking plugin. There are a large number of possible networking plugins for Kubernetes. SLATE clusters generally use Calico, although other options should work as well.

To install Calico, you will simply need to apply the appropriate Kubernetes manifests beginning with the operator:


    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/tigera-operator.yaml
    

If you haven’t changed the default IP range then create the boilerplate custom resources manifest:


    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/custom-resources.yaml
    

If you have changed the IP range to anything other than `192.168.0.0/16` in the `kubeadm init` command above, you will need to first download the boilerplate [custom-resources.yaml](https://github.com/projectcalico/calico/blob/master/manifests/custom-resources.yaml) file from Project Calico on GitHub, then update its IP range under `spec/calicoNetwork/ipPools/blockSize` and `CIDR`. Finally, create the custom resources manifest:


    kubectl create -f /path/to/custom-resources.yaml
    

After approximately five minutes, your master node should be ready. You can check with `kubectl get nodes`:


    [root@your-node ~]# kubectl get nodes
    NAME                           STATUS   ROLES                  AGE     VERSION
    your-node.your-domain.edu      Ready    control-plane        2m50s   v1.28.0
    

Load Balancer/MetalLB
---------------------

When using the Kubernetes service type of Load Balancer, [MetalLB](https://metallb.org/) will attach an IP address to the service from a pool of address ranges you specify in the MetalLB configuration. The SLATE team usually uses MetalLB to automatically attach public IP addresses to a service, but it can be used with RFC1918 IP addresses as well.

Installing and configuring MetalLB is done in three steps: installation, configuration, and advertisement. (There is a fourth step if you are using virtual machines managed by OpenStack.)

### Step 1: Installation

Run this command to install MetalLB:


    METALLB_VERSION=0.13.12 && \
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v${METALLB_VERSION}/config/manifests/metallb-native.yaml
    

### Step 2: Configuration

The following command creates an IP Address Pool file. Replace the addresses in this IPAddressPool configuration with the available IP addresses for your cluster. As noted in the example, you can specify a range with a CIDR or a hyphen. You may configure more than one range of IP addresses. The example configures two ranges (make sure to remove the second range if you only need one).


    cat <<EOF > /tmp/metallb-ipaddrpool.yml
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: first-pool
      namespace: metallb-system
    spec:
      addresses:
      - 192.168.10.0/24
      - 192.168.9.1-192.168.9.5
    EOF
    

You can now apply the configuration file to Kubernetes:


    kubectl create -f /tmp/metallb-ipaddrpool.yml
    

### Step 3: Advertisement

Once configured, we need to tell MetalLB how to advertise the IP addresses in the pool. In this case we are configuring the pool to use Layer 2 advertisement. [MetalLB also supports BGP](https://metallb.org/configuration/_advanced_bgp_configuration/).


    cat <<EOF > /tmp/metallb-ipaddrpool-advert.yml
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
      name: example
      namespace: metallb-system
    spec:
      ipAddressPools:
      - first-pool
    EOF
    

Apply the advertisement file to Kubernetes:


    kubectl create -f /tmp/metallb-ipaddrpool-advert.yml
    

### Step 4 (if you are running VMs on OpenStack): MetalLB on OpenStack

The [MetalLB documentation](https://metallb.universe.tf/faq/#is-metallb-working-on-openstack) notes the following for OpenStack managed virtual machines:

You can run a Kubernetes cluster on OpenStack VMs, and use MetalLB as the load balancer. However, you have to disable OpenStack’s ARP spoofing protection if you want to use L2 mode. You must disable it on all the VMs that are running Kubernetes.

By design, MetalLB’s L2 mode looks like an ARP spoofing attempt to OpenStack, because we’re announcing IP addresses that OpenStack doesn’t know about. There’s currently no way to make OpenStack cooperate with MetalLB here, so we have to turn off the spoofing protection entirely.

