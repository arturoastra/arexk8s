
### **Step 1: Download and Install Helm**

1. **Download the Helm installation script**:  
   Run the following command to download the Helm installation script:

    ```bash
    curl -O https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    ```

2. **Execute the Helm installation script**:  
   Run the downloaded script to install Helm:

    ```bash
    bash ./get-helm-3
    ```

3. **Verify Helm installation**:  
   Once installed, check the Helm version to verify the installation:

    ```bash
    helm version
    ```

---

### **Step 2: Install the MetalLB Chart using Helm**

1. **Add the MetalLB Helm repository**:  
   Add the MetalLB Helm repository using the following command:

    ```bash
    helm repo add metallb https://metallb.github.io/metallb
    ```

2. **Update the Helm repositories**:  
   Update Helm to ensure you have the latest version of the MetalLB chart:

    ```bash
    helm repo update
    ```

---

### **Step 3: Create and Configure the `values.yaml` File for MetalLB**

1. **Create the `values.yaml` file**:  
   Create a file named `values.yaml` with the following content to define an IP pool and the L2 advertisement configuration:

    ```yaml
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: single-ip-pool
    spec:
      addresses:
      - 94.72.121.89/32  # External IP to use
    ---
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
      name: single-ip-advertisement
    ```

---

### **Step 4: Apply MetalLB Configuration using Helm**

1. **Install MetalLB with the `values.yaml` file**:  
   Use Helm to install MetalLB, specifying the custom `values.yaml` file:

    ```bash
    helm install metallb metallb/metallb -f values.yaml
    ```

2. **Verify the installation**:  
   Check if the MetalLB pods are running:

    ```bash
    kubectl get pods -n metallb-system
    ```

---

### **Step 5: Verify External IP Assignment**

To ensure that the external IP (`94.72.121.89`) is correctly assigned to your service, check the status of the `LoadBalancer` services:

```bash
kubectl get svc
```

You should see the external IP in the `EXTERNAL-IP` column of your service.

---

This structure ensures all steps are logically organized and easy to follow.
