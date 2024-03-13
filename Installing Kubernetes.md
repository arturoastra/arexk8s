Installing Kubernetes
=====================

  **Note:** SLATE currently supports Kubernetes v1.28.

The SLATE platform uses Kubernetes as its container orchestration system. This section we’ll install the base Kubernetes software components.

Kubernetes
----------

The Kubernetes repository can be added to the node in the usual way:



    export KUBE_VERSION=1.28 && \
    cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://pkgs.k8s.io/core:/stable:/v${KUBE_VERSION}/rpm/
    enabled=1
    gpgcheck=1
    gpgkey=https://pkgs.k8s.io/core:/stable:/v${KUBE_VERSION}/rpm/repodata/repomd.xml.key
    EOF
    

The Kubernetes install includes a few different pieces: `kubeadm`, `kubectl`, and `kubelet`.

*   `kubeadm` is a tool used to bootstrap Kubernetes clusters
*   `kubectl` is the command-line tool needed to interact with and control the cluster
*   `kubelet` is the system daemon that allows the Kubernetes api to control the cluster nodes

Install and enable these components:



    yum install -y kubeadm kubectl kubelet
    

Finish Up
---------

Finally, enable `kubelet`:



    systemctl enable --now kubelet
    

At this point the `kubelet` will be crash-looping as it has no configuration. That is okay for now.

