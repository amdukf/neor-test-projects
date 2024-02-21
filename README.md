# Report of the projects


## Installing Minikube and argocd with helm chart

### Installing Minikube
##### Prerequisites
- sudo apt update
- sudo apt upgrade -y
- sudo apt install curl wget apt-transport-https -y
- sudo reboot

##### Installing Docker due to [Official Docker Installation](https://docs.docker.com/engine/install/ubuntu/) 

##### Download and Install Minikube Binary
- curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
- sudo install minikube-linux-amd64 /usr/local/bin/minikube
##### At that point, I checked the version of Minikube installed by running the command below:
- minikube version

##### Install kubectl tool

- curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
- chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin
verify the installation by running the command:
- kubectl version --client -o yaml

##### Start Minikube Using Docker driver (Or we can start it without anything using minikube start)
- minikube start --driver docker (Spent a lot of time installing)
After that to view the status of minikube:
- minikube status
and for check the kubernetes version and pod status:
- kubectl get pods

### Installing argocd with helm chart

##### create a Helm "umbrella chart"
- mkdir -p charts/argo-cd
and after that we create the Charts.yaml and values.yaml and place this yamls in them:

Charts.yaml :
```yaml
apiVersion: v2
name: argo-cd
version: 1.0.0
dependencies:
  - name: argo-cd
    version: 5.46.8
    repository: https://argoproj.github.io/argo-helm

```

values.yaml:
```yaml
argo-cd:
  dex:
    enabled: false
  notifications:
    enabled: false
  applicationSet:
    enabled: false
  server:
    extraArgs:
      - --insecure
```
###### At first i use this extraArgs, but when i go to second project, i change the extraArgs and add the - --server-root-path=/ into it


##### Before we install our chart, we need to generate a Helm chart lock file for it
- helm repo add argo-cd https://argoproj.github.io/argo-helm
- helm dep update charts/argo-cd/

##### Installing our argocd helm chart
- helm install argo-cd charts/argo-cd/
After a minute all resources should have been deployed, we check it with:
- kubectl get pods

##### Accessing the argocd Web UI
- kubectl port-forward svc/argo-cd-argocd-server 8080:443 --address 0.0.0.0
0.0.0.0 is for getting access it from the outside of the virtual machine
##### generating password for UI
- kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
or we can do this with that method that we get the output of kubectl get secret with -o switch and get the yaml file and decode it into base64

---
## Installing Ingress for our argocd

##### At first, i created the ingress yaml file (manifest)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
spec:
  rules:
  - host: argocd.amir.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argo-cd-argocd-server
            port:
              number: 80
```

##### and after that i used <kubectl apply -f argocd-ingress.yaml> to apply Ingress Manifest and we check the ingress status with<kubectl get ingress>

