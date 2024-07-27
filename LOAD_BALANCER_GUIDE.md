# Making LoadBalancer work on your own kubernetes
- Ref Link: https://metallb.universe.tf/installation/
 
### Run
```
kubectl edit configmap -n kube-system kube-proxy
```
### and search and replace current values set:
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```
### Then apply:
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
kubectl apply -f - <<EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 192.168.50.112-192.168.50.116 # Put yours available ip range or CIDR notation ex.: 192.168.0.0/24
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default
EOF
```
