# eth_fullnode_example
Example that bootstraps and runs an eth node

Pre-requisites
* ansible >= 9.0.0 (older versions may work)
* kustomize >= 5.0.0 (older versions may work)
* kubectl >= 1.27.0

Mac Setup
```commandline
brew install ansible
brew install kustomize
brew install kubectl
```

Running the bootstrap
```commandline
# Set up passwordless ssh to your target machine
ssh-copy-id YOUR_USER@${YOUR_IP_ADDRESS}
git clone git@github.com:bizrad/eth_fullnode_example.git
cd eth_fullnode_example
sed -i 's/192.168.0.1/YOUR_IP_OR_HOSTNAME_HERE/g' inventory.yaml
ansible-playbook -i inventory.yaml ansible-host-bootstrap.yaml
```

### Access microk8s api from your local machine 
Set up port forwarding to the remote host and use kubectl
See: https://microk8s.io/docs/working-with-kubectl
You can pull the remote system kubectl config and use it locally
Generate it by running `microk8s config` on the remote.
Change out the `server` hostname with localhost and the port you are forwarding
Forward a port from k8s over ssh
```commandline
export YOUR_IP_ADDRESS=192.168.0.1
ssh -L 16443:localhost:16443 root@${YOUR_IP_ADDRESS}
```

### View ArgoCD web UI with default creds
```shell
# Get initial generated admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
# Start port forwarding
kubectl port-forward -n argocd service/argocd-server 8080:80 
```
Open in web browser:

Username: admin

Password: (see above)

http://localhost:8080

### View Grafana Dashboard web UI with default creds
```shell
# Start port forwarding
kubectl port-forward -n observe service/kube-prometheus-stack-grafana 8080:80 
```
Open in web browser:

Username: admin

Password: prom-operator

http://localhost:8080

http://localhost:8080/d/M2TT9Su7z/ethereum-metrics-exporter-single

http://localhost:8080/d/ddgd7oddnhreob/node-exporter-nodes?orgId=1&refresh=30s
