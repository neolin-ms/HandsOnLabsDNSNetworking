# Hands-On Labs DNS Networking - Scenario 4

## Step.1 Creat namespace using below command:
```bash
kubectl create namespace bar 
```
Example output:<br>

## Step.2 Create a coredns-rewrite-podsvc-example.yaml file and copy in the following example YAML:
```bash
nano coredns-rewrite-podsvc-example.yaml
```
## Step.3 To create th Pod,Service shown above, run the following command:
```bash
kubectl apply -f coredns-rewrite-example.yaml --namespace bar
```
Example output:<br>
## Step.4 Check the Pod,Service and note the Service IP, run the following command:
```bash
kubectl get -n bar pods,svc 
```
Example output:<br>

## Step.5 Connect to the pod, run the following command:
```bash
kubectl exec -it [POD_NAME] -n bar -- /bin/sh
```
Example Output:<br>

## Step.6 Install the dig tool on the pod, run the following command:
```bash
apt-get update && apt-get install dnsutils -y
```
## Step.7 Check the DNS resolution for www.microsoft.com, run the following command: We find that get an external IP.
```bash
dig www.microsoft.com.a +short
```
Example output:<br>

## Step.8 Create a coredns-rewrite-configmap-example.yaml and copy in the following example YAML:
```bash
nano coredns-rewrite-configmap-example.yaml
```
## Step.9 Apply configmap to your cluster and delete current coredns pods.
```bash
kubectl apply -f coredns-rewrite-configmap-example.yaml
kubectl delete pods -n kube-system -l k8s-app=kube-dns
```
Example output:<br>
## Step.10 Access your pod(app1) and check if DNS resolution for www.microsoft.com gets an internal IP(with your service CIDR scope) list:
```bash
dig www.microsoft.com.a +short
```
Example output:<br>
