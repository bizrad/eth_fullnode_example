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

# Decisions
* Bootstraping the node with ansible as this is a good way to ensure a repeatable setup.
* Deploying the software with kubernetes as this is a fun relevant example and I'm familiar with it.
* using `ufw` firewall as this is the default tool for ubuntu
* using microk8s as this is a lightweight k8s distribution that is easy to install and use

## Ethereum Node Software
Using [geth aka go-ethereum](https://github.com/ethereum/go-ethereum) because it has fewer issues, more commits, and more stars on github than Nethermind. 
This is an arbitrary decision without much thought put into it, but it indicates geth might be more mature.
Also went with lighthouse as it seemed the most popular and less likely to have issues. 

# Ways to improve
* In a production environment with different monitoring available on a host level
[microk8s in strict mode](https://microk8s.io/docs/strict-confinement) might be interesting for sandboxing.
* `ufw` might not be the best firewall, evaluate if other options are better.
* review microk8s security and hardening options
* Use something like Traefik for ingress to allow more flexibility with alternate ports

# Shortcuts
These were done to not commit the ip address to a public repo or work around issues
* nginx ingress is manually configured due to needing to forward a non-standard port
* ingress object is manually deployed to avoid exposing data in a public repo
* secrete manually deployed as we don't have a secret manager
* manually made ufw rules based on microk8s docs, these open things up a lot