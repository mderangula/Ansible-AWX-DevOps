# Ansible AWX DevOps

A practical lab repository for learning infrastructure automation with Ansible,
AWX, Docker, Terraform, Kubernetes, Argo CD, and GitHub Actions.

## Repository layout

```text
ansible/       Inventories, playbooks, roles, and collection requirements
argocd/        Argo CD project, application, and bootstrap instructions
gitops/        Kubernetes manifests managed by Argo CD
terraform/     Configurable NGINX deployment for a Kubernetes lab cluster
devops-labs/   Container and AWX deployment labs
```

## Prerequisites

Install only the tools required for the lab you want to run:

- Ansible and Python
- Docker with Docker Compose
- Terraform 1.7 or newer
- kubectl and a Kubernetes cluster such as Minikube
- Argo CD CLI for GitOps operations

Review all inventory addresses, credentials, Kubernetes contexts, and variables
before running commands against your own systems.

## Ansible

Install the required collection and run the local connection test:

```bash
ansible-galaxy collection install -r ansible/requirements.yml
ansible-playbook -i ansible/inventory.ini ansible/playbooks/test-connection.yml
```

Use syntax and check mode before applying a host-changing playbook:

```bash
ansible-playbook -i ansible/inventory.ini PLAYBOOK --syntax-check
ansible-playbook -i ansible/inventory.ini PLAYBOOK --check --diff
```

## Docker lab

The container lab runs a small Flask application through Gunicorn as a non-root
user. It includes a health endpoint, read-only filesystem, and localhost-only
port binding.

```bash
docker compose -f devops-labs/lab1-docker/docker-compose.yml up --build
curl http://127.0.0.1:8080/health
docker compose -f devops-labs/lab1-docker/docker-compose.yml down
```

## Terraform lab

Terraform uses the selected kubeconfig context to create a namespace, a
two-replica NGINX Deployment, and a private ClusterIP Service.

```bash
terraform -chdir=terraform init
terraform -chdir=terraform fmt -check
terraform -chdir=terraform validate
terraform -chdir=terraform plan
```

The default context is `minikube`. Override it when needed:

```bash
terraform -chdir=terraform plan -var='kubeconfig_context=my-context'
```

## AWX lab

The AWX Kustomize configuration installs the pinned AWX Operator and creates the
lab AWX instance in namespace `awx`:

```bash
kubectl apply -k devops-labs/lab3-awx
kubectl -n awx get pods
```

The NodePort service configuration is intended for a local lab. Add persistent
storage, trusted TLS, ingress, backups, and identity integration before using
AWX in a production environment.

## Argo CD and GitHub Actions

Bootstrap the persistent server service, project, and application:

```bash
kubectl apply -k argocd
argocd app get hello-world
```

Changes under `argocd/` or `gitops/` trigger the `GitOps Delivery` workflow.
Pull requests validate manifests. Changes on `main` synchronize the exact commit
through Argo CD and publish status, history, resources, and recent pod logs.

Create a GitHub environment named `production` with these secrets:

- `ARGOCD_SERVER`
- `ARGOCD_AUTH_TOKEN`

See [argocd/README.md](argocd/README.md) for bootstrap and networking details.

## Important local-file safety

This repository intentionally has no `.gitignore`. Before staging changes,
check for Terraform state, `.terraform/`, `.env`, logs, editor files, Python
caches, and other generated or sensitive files. Never commit credentials or
Terraform state.

## License

This project is available under the MIT License.
