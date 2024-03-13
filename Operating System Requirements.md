Operating System Requirements
=============================

Disable SELinux
---------------

First, you will need to disable SELinux as this generally conflicts with Kubernetes:

```shell
setenforce 0 && \
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

  **Note:** If you wish to retain the SELinux logging, you can alternatively use **permissive** mode rather than disabling it entirely.

Disable swap
------------

Swap must be disabled for Kubernetes to run effectively. Swap is typically enabled in a default CentOS 7 installation where automatic partitioning has been selected. To disable swap:

```shell
swapoff -a && \
sed -e '/swap/s/^/#/g' -i /etc/fstab
```

Disable firewalld
-----------------

In order to properly communicate with other devices within the cluster, `firewalld` must be disabled:

```shell
systemctl disable --now firewalld
```

Disable root login over SSH
---------------------------

  **Important:** This is highly recommended for security reasons.

Optionally disable `root` login over SSH.

```shell
sed -i --follow-symlinks 's/#PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
```  

Use iptables for Bridged Network Traffic
----------------------------------------

  **Note:** This step is only necessary for EL7 and EL8 hosts.

Ensure that bridged network traffic goes through `iptables`.

```shell
 cat <<EOF >  /etc/sysctl.d/iptables-bridge.conf
 EOF
 sysctl --system
``` 

Enable routing
--------------

```shell
cat <<EOF >  /etc/sysctl.d/ip-forward.conf
net.ipv4.ip_forward = 1 
EOF
sysctl --system
```
