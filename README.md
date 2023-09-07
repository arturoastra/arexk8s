

## Instructions

### Display Deployment Status for the "arex" Namespace

```shell
sudo su
```

```shell
kubectl get deployments -n arex
```

Expected Output:

```shell
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
arexapi         1/1     1            1           12d
prestodb-arex   1/1     1            1           12d
```

### Restart Deployments in the "arex" Namespace

```shell
kubectl -n arex rollout restart deployment prestodb-arex
```

### Stop Port Forwarding

To stop all port forwarding processes, run the following command:

```shell
for p in $(ps -ef | grep 'port-forward' | grep -v 'grep' | awk '{print $2}') ; do kill $p ; done
```

### Restart Port Forwarding

To restart port forwarding for services in the "arex" namespace, use the following commands:

```shell
nohup kubectl port-forward svc/arexapi -n arex 80:80 --address=0.0.0.0 &>/dev/null &
nohup kubectl port-forward svc/prestodb-arex -n arex 8080:8080 --address=0.0.0.0 &>/dev/null &
```

### Validate That Port Forwarding Is Running

To check if port forwarding processes are running, use the following command:

```shell
ps -ef | grep 'port-forward' | grep -v 'grep'
```

### Access the API Documentation

Access the API documentation using the following URL:

```shell
http://ec2-3-13-25-109.us-east-2.compute.amazonaws.com/docs
```

### Access to arex-presto catalog

```shell
kubectl exec -it $(kubectl get pods -n arex | grep prestodb-arex | awk '{print $1}') -c prestodb -n arex -- bash
```
#### Move into the catalog folder

```shell
cd /etc/presto/catalog/
```


These revised instructions provide clear titles and improved formatting to help users follow the steps more easily.
```

