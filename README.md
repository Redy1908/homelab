## Homelab Setup

My personal homelab on **Talos Linux** powered by **Kubernetes**. Fully automated with **Ansible** and managed with GitOps (**FluxCD**).

## Deployed Services

- **Homepage**: Personal homepage and dashboard.
- **Pi-hole**: Network-wide ad blocking and DNS server.
- **Nginx Proxy Manager**: Reverse proxy for exposing services.
- **Actual Budget**: Personal finance management.
- **Tailscale**: Secure remote access to the cluster.
- **MetalLB**: Bare metal load balancer.
- **n8n**: Workflow automation tool.
- **Cloudflared**: Cloudflare Tunnel for secure access.

## Development Environment

This project is configured with a **Dev Container**. All tools (`kubectl`, `talosctl`, `flux`, `ansible`, `sops`, `age`) are pre-installed and configured.

## Cluster Setup

Delete the existing encrypted secrets:

```bash
rm -f talos/secrets/secret.yaml
rm -f apps/pi-hole/secret.yaml
```

Create the secret for Pi-hole `apps/pi-hole/secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: pihole-secret
    namespace: pihole
stringData:
    password: YOUR_PIHOLE_PASSWORD
```

> Ansible will create the Talos secret `talos/secrets/secret.yaml` during the playbook execution.

Generate an `age` key pair for SOPS encryption:

```bash
age-keygen -o key.txt
export SOPS_AGE_KEY_FILE=key.txt
```

Edit the file `.sops.yaml` to add your age public key under the `age` section.

Encrypt the pihole secret

```bash
sops -e -i apps/pi-hole/secret.yaml
```

Export the necessary environment variables used by the Ansible playbook:

```bash
export GITHUB_USER="GITHUB_USERNAME"
export GITHUB_REPO="GITHUB_REPO"
export GITHUB_TOKEN="GITHUB_TOKEN"
export CLUSTER_NAME="homelab"
export NODE_IP="NODE_IP"
```

Run the ansible playbook to set up the cluster.

```bash
ansible-playbook ansible/talos/initialize.yaml
```

Encrypt the Talos secret

```bash
sops -e -i talos/secrets/secret.yaml
```

FluxCD will automatically deploy the applications defined in the `clusters/homelab/apps` directory.

> **Note**: This is a single node configuration. In case make sure to edit the file `talos/patches/patch.yaml` to set the correct installation disk.

## Patch The Cluster Configuration (Optional)

Decript the Talos secret

```bash
sops -d -i talos/secrets/secret.yaml
```

Use the dedicated Ansible playbook to patch the cluster configuration.

```bash
ansible-playbook ansible/talos/patch.yaml
```
> Will export the current Cluster config using also the file `talos/patches/patch.yaml` in `talos/config` and will apply it to the cluster.

Encrypt the Talos secret

```bash
sops -e -i talos/secrets/secret.yaml
```

## Recover the Cluster configuration (Optional)

Decript the Talos secret

```bash
sops -d -i talos/secrets/secret.yaml
```

Use the dedicated Ansible playbook to recover the cluster configuration.

```bash
ansible-playbook ansible/talos/recover.yaml
```
> Will export the current Cluster config including kubeconfig in `talos/config`.

Encrypt the Talos secret

```bash
sops -e -i talos/secrets/secret.yaml
```

## VPS Setup with Ansible (Headscale)

To install and configure Headscale on your VPS using Ansible:

1. Export the required environment variables:

```bash
export HEADSCALE_URL="https://headscale.example.com"
export HEADSCALE_BASE_DOMAIN="example.com"
export TLS_LETSENCRYPT_HOSTNAME="headscale.example.com"
export VPS_HOST="your.vps.hostname.or.ip"
export VPS_USER="your_ssh_user"
```

2. Run the dedicated playbook using the provided inventory:

```bash
ansible-playbook ansible/vps/install-headscale.yaml -i ansible/inventory.ini
```

- Il file `ansible/inventory.ini` usa le variabili d'ambiente VPS_HOST e VPS_USER per configurare dinamicamente l'host e l'utente SSH:

```ini
[vps]
vps1 ansible_host={{ lookup('env', 'VPS_HOST') }} ansible_user={{ lookup('env', 'VPS_USER') }}
```

- The playbook will:
  - Download and install Headscale
  - Render the config file from `ansible/vps/config.yaml.j2` using the exported variables
  - Enable and start the Headscale service
