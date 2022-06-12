# Hands-On Labs DNS Networking - Scenario 4

## Step.1 Creat namespace using below command:
```bash
kubectl create namespace bar 
kubectl get namespaces
```
Example output:<br>
![Step01](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/Pics/Step01.png)
## Step.2 Create a coredns-rewrite-podsvc-example.yaml file and copy in the following example YAML. Here is reference [YAML file](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/YAML/coredns-rewrite-podsvc-example.yaml).
```bash
nano coredns-rewrite-podsvc-example.yaml
```
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: aks-helloworld-one
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"
---
apiVersion: v1
kind: Service
metadata:
  name: www  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: app1
```
## Step.3 To create th Pod,Service shown above, run the following command:
```bash
kubectl apply -f coredns-rewrite-podsvc-example.yaml --namespace bar
```
Example output:<br>
![Step03](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/Pics/Step03.png)
## Step.4 Check the Pod,Service and note the Service IP(CLUSTER-IP), run the following command:
```bash
kubectl get -n bar pods,svc 
```
Example output:<br>
![Step04](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/Pics/Step04.png)
## Step.5 Connect to the pod(app1), run the following command:
```bash
kubectl exec -it [POD_NAME] -n bar -- /bin/sh
```
Example output:<br>
![Step05](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/Pics/Step05.png)
## Step.6 Install the dig tool on the pod, run the following command:
```bash
apt-get update && apt-get install dnsutils -y
```
## Step.7 Check the DNS resolution for www.microsoft.com, run the following command. We find that get an external IP.
```bash
dig www.microsoft.com.a +short
```
Example output:<br>
![Step07](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/Pics/Step07.png)
## Step.8 Create a coredns-rewrite-configmap-example.yaml and copy in the following example YAML. Please replace the [Your_DNS_Service_IP] to Service IP of Step04. Here is reference [YAML file](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/YAML/coredns-rewrite-configmap-example.yaml).
```bash
nano coredns-rewrite-configmap-example.yaml
```
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  test.server: |
    microsoft.com:53 {
        errors
        cache 30
        rewrite name substring microsoft.com bar.svc.cluster.local
        forward . [Your_DNS_Service_IP] # # you can redirect this to a specific DNS server such as 10.1.0.20
    }
```
## Step.9 Apply configmap to your cluster and delete current coredns pods.
```bash
kubectl apply -f coredns-rewrite-configmap-example.yaml
kubectl delete pods -n kube-system -l k8s-app=kube-dns
```
Example output:<br>
![Step09](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/Pics/Step09.png)
## Step.10 Access your pod(app1) and check if DNS resolution for www.microsoft.com gets an internal IP(with your service CIDR scope) list:
```bash
dig www.microsoft.com.a +short
```
Example output:<br>
![Step10](https://github.com/neolin-ms/HandsOnLabsDNSNetworking/blob/main/Pics/Step10.png)
## Step.11 Clean up resources.
```bash
az group delete -n [AKS_Resource_Group] --no-wait --yes
az group delete -n [AKS_Nodes_Resource_Group] --no-wait --yes
```
