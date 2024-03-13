Installing Containerd
=====================

The SLATE platform uses `containerd` as the container run-time. Complete the following steps to install and configure `containerd` for your cluster.

Load Kernel Modules
-------------------

Specify and load the following kernel module dependencies:



    cat <<EOF | tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF
    

Copy

    modprobe overlay && \
    modprobe br_netfilter
    

Add Yum Repo
------------

Install the `yum-config-manager` tool if not already present:

Copy

    yum install yum-utils -y
    

Add the stable Docker Community Edition repository to `yum`:

Copy

    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    

Install Containerd
------------------

Install the latest version of `containerd`:

Copy

    yum install containerd.io -y
    

Configure cgroups
-----------------

Configure the `systemd` `cgroup` driver:

Copy

    CONTAINDERD_CONFIG_PATH=/etc/containerd/config.toml && \
    rm "${CONTAINDERD_CONFIG_PATH}" && \
    containerd config default > "${CONTAINDERD_CONFIG_PATH}" && \
    sed -i "s/SystemdCgroup = false/SystemdCgroup = true/g"  "${CONTAINDERD_CONFIG_PATH}"
    

Finish Up
---------

Finally, enable `containerd` and apply the changes:

Copy

    systemctl enable --now containerd && \
    systemctl restart containerd
    

[Next Page »](/docs/cluster/manual/kubernetes.html)
