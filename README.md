# GitOps CI/CD Pipeline with GitHub Actions & ArgoCD

In this project, I implemented a GitOps-based CI/CD pipeline using GitHub Actions and ArgoCD (for Kubernetes management). The pipeline is built around a simple Python application using Flask.

The main goal of this project is to experiment with and learn GitOps CI/CD practices. While the application itself is minimal, the approach can easily be extended to more complex projects using the same pipeline design.

---

# Project Structure

```
.
├── app.py              # Application source code
├── Dockerfile          # Docker image definition
├── requirements.txt    # Python dependencies
└── README.md
```


# Setup

In this project, a **Minikube Kubernetes cluster** is deployed with Docker as the underlying runtime.

Instead of using a local environment, I chose to deploy everything on a **virtual machine provisioned with Terraform** on the **VirtualData cloud (CNRS)**. The required tools are installed during VM provisioning using a **cloud-init configuration integrated into Terraform**.

If you want to reproduce this setup, here is what is installed at VM initialization with cloud-init (AlmaLinux 9.x):

```bash
# Docker installation
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl enable docker
systemctl start docker

# kubectl installation
curl -LO https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm -f kubectl

# minikube installation
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
rm -f minikube-linux-amd64

# Create a non-root user for Minikube (IMPORTANT)
useradd -m devops || true
usermod -aG docker devops

# Start Minikube as non-root user
su - devops -c "minikube start --driver=docker"
su - devops -c "minikube addons enable ingress"

# Configure kubectl for root user
mkdir -p /root/.kube
cp /home/devops/.kube/config /root/.kube/config || true
export KUBECONFIG=/root/.kube/config

# --- ArgoCD installation ---
su - devops -c "kubectl create namespace argocd || true"
su - devops -c "kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml"

sleep 60

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

## Access ArgoCD

After deployment, in vm:

```bash
su - devpos -c "kubectl get svc -n argocd"
```

Then retrieve the password:

```bash 
su - devpos -c "kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d"
```