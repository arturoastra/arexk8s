Installing Containerd
=====================

### Step 1: Turn Off Swap (all nodes)
To permanently disable swap, we need to comment out the swap partition in `/etc/fstab`:

```bash
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Step 2: Set SELinux in permissive mode (effectively disabling it)
Temporarily disable SELinux and modify the configuration file to permanently disable it:

```bash
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

Verify the SELinux status:

```bash
cat /etc/selinux/config
```

### Step 3: Load required modules and configure them to load at boot time
Load the required modules:

```bash
modprobe overlay
modprobe br_netfilter
```

Add the modules to load at boot:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

Set other prerequisites:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

Apply the changes without restarting:

```bash
sysctl --system
```

Stop the firewall on the master node:

```bash
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```

### Step 4: Install containerd
Add the official Docker repository:

```bash
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf update -y
```

Install the `containerd` package:

```bash
dnf install -y containerd.io
```

Create a configuration file for containerd and set it as default:

```bash
mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Set the `cgroupdriver` to `systemd`:

Edit the `/etc/containerd/config.toml` file and locate the following section:

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
```

Change the value of `SystemdCgroup` to `true`. The section should look like this:

```
CONTAINDERD_CONFIG_PATH=/etc/containerd/config.toml && \
rm "${CONTAINDERD_CONFIG_PATH}" && \
containerd config default > "${CONTAINDERD_CONFIG_PATH}" && \
sed -i "s/SystemdCgroup = false/SystemdCgroup = true/g"  "${CONTAINDERD_CONFIG_PATH}"
```

Restart `containerd` to apply the changes:

```bash
systemctl restart containerd
systemctl enable containerd
systemctl status containerd
```

Verify that `containerd` is running:

```bash
ps -ef | grep containerd
```

### Step 5: Install the following packages (on all nodes)

```bash
yum install -y wget
yum install -y vim-enhanced
yum install -y git
yum install -y curl
```
