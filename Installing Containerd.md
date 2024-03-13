Installing Containerd
=====================

Load Kernel Modules
-------------------

Specify and load the following kernel module dependencies:



    cat <<EOF | tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF
    


    modprobe overlay && \
    modprobe br_netfilter
    

Add Yum Repo
------------

Install the `yum-config-manager` tool if not already present:


    yum install yum-utils -y
    

Add the stable Docker Community Edition repository to `yum`:


    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    

Install Containerd
------------------

Install the latest version of `containerd`:


    yum install containerd.io -y
    

Configure cgroups
-----------------

Configure the `systemd` `cgroup` driver:


    CONTAINDERD_CONFIG_PATH=/etc/containerd/config.toml && \
    rm "${CONTAINDERD_CONFIG_PATH}" && \
    containerd config default > "${CONTAINDERD_CONFIG_PATH}" && \
    sed -i "s/SystemdCgroup = false/SystemdCgroup = true/g"  "${CONTAINDERD_CONFIG_PATH}"
    

Finish Up
---------

Finally, enable `containerd` and apply the changes:


    systemctl enable --now containerd && \
    systemctl restart containerd
    
