
# Kube-vip
## 1. step: On-Premises (kube-vip-cloud-controller)

### Install the kube-vip Cloud Provider
```sh
kubectl apply -f https://raw.githubusercontent.com/kube-vip/kube-vip-cloud-provider/main/manifest/kube-vip-cloud-controller.yaml
```

### Create a global CIDR or IP Range
* CIDR
```sh
kubectl create configmap -n kube-system kubevip --from-literal cidr-global=10.100.200.220/29
```
* or ip range
```sh
kubectl create configmap -n kube-system kubevip --from-literal range-global=10.100.200.220-10.100.200.230
```
### expose example service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
  selector:
    app: nginx
  type: LoadBalancer
  # specific ip (optional)
  loadBalancerIP: "10.100.200.221"
```

## 2. step: kube-vip as HA (daemonset), created on one of the cluster node
### Create the RBAC settings
```sh
kubectl apply -f https://kube-vip.io/manifests/rbac.yaml
```
### Generating a Manifest
* Set configuration details
```sh
# Set the VIP address to be used for the control plane
export VIP=10.100.200.210

# Set the INTERFACE name which will announce the VIP
export INTERFACE=ens18

# Get the latest version of the kube-vi
export KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")

# For containerd, run the below command:
alias kube-vip="ctr image pull ghcr.io/kube-vip/kube-vip:$KVVERSION; ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"

# ARP Example for DaemonSet
```sh
kube-vip manifest daemonset \
    --interface $INTERFACE \
    --address $VIP \
    --inCluster \
    --taint \
    --controlplane \
    --services \
    --arp \
    --leaderElection | tee kube-vip-daemonset.yaml
```
