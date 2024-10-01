Installing Kubernetes
=====================

### Step 1: Install Kubernetes Components

1. **Add Kubernetes Repository**  
   First, add the Kubernetes repository to your package manager:

   ```bash
   cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
   enabled=1
   gpgcheck=1
   gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
   exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
   EOF
   ```

2. **Install Kubernetes Packages**  
   Install the Kubernetes components (`kubelet`, `kubeadm`, and `kubectl`):

   ```bash
   dnf makecache; dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
   ```

3. **Start and Enable kubelet Service**  
   Start and enable the kubelet service:

   ```bash
   systemctl enable --now kubelet.service
   ```

### Step 2: Initializing Kubernetes Control Plane

1. **Pull Required Images**  
   Before initializing the control plane, pull the necessary images:

   ```bash
   sudo kubeadm config images pull
   ```

2. **Initialize the Control Plane**  
   Initialize Kubernetes control plane on the master node:

   ```bash
   sudo kubeadm init --pod-network-cidr=10.244.0.0/16
   ```

3. **Configure kubeconfig**  
   Set up kubeconfig file for the cluster:

   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

### Step 3: Deploy Pod Network

1. **Deploy Tigera Operator for Calico**  
   Apply the Tigera Operator for Calico networking:

   ```bash
   kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
   ```

2. **Download Custom Calico Resources**  
   Fetch the Calico resources manifest using either `curl` or `wget`:

   ```bash
   curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
   ```

3. **Adjust CIDR and Apply Calico**  
   Modify the CIDR in the file and apply:

   ```bash
   sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.244.0.0\/16/g' custom-resources.yaml
   kubectl create -f custom-resources.yaml
   ```

### Step 4: Join Worker Nodes

1. **Generate Join Command on Master Node**  
   Run this command on the master node to get the join command:

   ```bash
   sudo kubeadm token create --print-join-command
   ```

2. **Run Join Command on Worker Nodes**  
   Use the generated command on each worker node:

   ```bash
   sudo kubeadm join <MASTER_IP>:<MASTER_PORT> --token <TOKEN> --discovery-token-ca-cert-hash <DISCOVERY_TOKEN_CA_CERT_HASH>
   ```

3. **Verify Worker Node Join**  
   Back on the master node, verify the status of worker nodes:

   ```bash
   kubectl get nodes
   ```

These are the complete instructions with the steps reordered starting from "Install Kubernetes Components".
