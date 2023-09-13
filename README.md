# Setup using Kubdeadm
## Commands to be run on both nodes.
1. Turn swapoff and install docker.
```bash 
sudo su
swapoff -a
apt update -y
apt install docker.io -y
```
2. Enable docker.
```bash 
systemctl start docker
systemctl enable docker
curl -fsSL "https://packages.cloud.google.com/apt/doc/apt-key.gpg" | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg
echo 'deb https://packages.cloud.google.com/apt kubernetes-xenial main' > /etc/apt/sources.list.d/kubernetes.list
```
3. Install kubeadm, kubelet and kubectl.
```bash 
apt update -y
apt install kubeadm=1.28.0-00 kubectl=1.28.0-00 kubelet=1.28.0-00 -y
```

## Master node Cdmmands
1. The kubeadm init command is used to initialize a new Kubernetes control-plane node. The kubeadm init command is used to initialize a new Kubernetes control-plane node. 
Note: Dont’t forgot to open port 6443 on master node.
 ```bash 
sudo su
kubeadm init
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

## Worker Node Commands
Note(IMPORTANT): Paste the Join command on worker node and append `--v=5` at end
 ```bash 
sudo su
kubeadm reset pre-flight checks
kubeadm join [.....] --v=5
```
## Create namespaces
1. Create namespaces. 
 ```bash 
kubectl create namespace staging
kubectl create namespace production
```
## Deliverables
Create cluster with two worker nodes using Kubeadm.
# Tekton Installation
1. Install the Tekton using the below command.
```bash 
kubectl apply --filename \
https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```
2. Create a Kubernetes Secret, docker-credentials.yaml with your credentials:
```bash 
apiVersion: v1
kind: Secret
metadata:
  name: docker-credentials
data:
  config.json: efuJAmF1...
Note: The value of config.json is the base64-encoded ~/.docker/config.json file. You can get this data with the following command:
cat ~/.docker/config.json | base64 -w0
```

3. Use the following commands to apply secrets, tasks(git clone and kaniko) and pipeline with pipelinerun.
```bash 
kubectl apply -f docker-credentials.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/kaniko/0.6/kaniko.yaml
kubectl apply -f pipeline.yaml
kubectl create -f pipelinerun-frontend.yaml
```
NOTE: Last Command creates a PipelineRun with a unique name each time:
pipelinerun.tekton.dev/clone-build-push-run-4kgjr created

4. Command to check the logs.
```bash 
tkn pipelinerun logs clone-build-push-run-4kgjr -f
```
5. Now go to DockerHub to verify. 
NOTE: Don't forgot to refresh the dockerhub repo page. 


## Deliverables
Acceptance criteria for the task are as follows: 
* Image should be uploaded on dockerhub.

# Configure AgroCD
The task is to configure agroCD on your cluster.
## Steps
1. Create a namespace and apply manifest installation from github repo.
```bash 
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.5.8/manifests/install.yaml
```
2. Verify it and make sure all the pods are running before moving to next step.
```bash 
kubectl get all -n argocd
```
3. In order to access the web UI you have to do port forwarding.
```bash 
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
4. Admin will be the username, now use command to get the password and copy the password.
```bash 
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
## Deliverables
Acceptance criteria for the task are as follows: 
* AgroCD will be configured.

# Configure Ingress
The task is to configure ingress on your cluster.
## Steps
1. Add ngrok controller and its credentials.
```bash 
helm repo add ngrok https://ngrok.github.io/kubernetes-ingress-controller
export NGROK_AUTHTOKEN=[AUTHTOKEN]
export NGROK_API_KEY=[API_KEY]
helm install ngrok-ingress-controller ngrok/kubernetes-ingress-controller \
  --namespace ngrok-ingress-controller \
  --create-namespace \
  --set credentials.apiKey=$NGROK_API_KEY \
  --set credentials.authtoken=$NGROK_AUTHTOKEN

```
2. Apply the resource file and verify it.
```bash 
kubectl  get pods,domains,httpsedges,tunnels -n ngrok-ingress-controller

```


## Deliverables
Acceptance criteria for the task are as follows: 
* Ingress should be configured.
