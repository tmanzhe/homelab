# homelab

Self hosted kubernetes homelab, built from bare metal up. primary purpose of this is to reuse my ewaste, solidify my learning on networking, infra as code, prod style k8s operation. learned alot from prev internship saws our server room hosting our clusters and got inspired. 

## Current architecture

- **RKE2** was chosen over k3s specifically because it's closer to prod grade Kubernetes (etcd-backed, CIS-hardened by default) rather than k3s's lightweigh design.
- **TrueNAS SCALE** runs the infrastructure that has to survive independently of the cluster — DNS (Pi-hole), remote access (Tailscale subnet router), and eventually the NFS storage backend. It is deliberately **not** a Kubernetes node itself, to avoid a circular dependency where a cluster failure takes down the DNS/VPN needed to fix it.
- **Ansible** drives the cluster bootstrap: idempotent, role-based automation rather than a set of manual shell commands. See `ansible/roles/rke2/` for the actual logic.
- Networking is intentionally double NAT'd (ISP router → a second router in full Router Mode) to create an isolated lab subnet for practicing real subnetting/segmentation.

## Repo structure

```
ansible/
├── ansible.cfg              # Ansible tool settings
├── inventory.ini.example    # Copy to inventory.ini and fill in real values (gitignored)
├── playbook.yaml            # Entry point: bootstraps the first RKE2 server
├── group_vars/
│   └── all.yml              # Shared vars (pinned RKE2 version)
└── roles/
    └── rke2/                # Reusable install/configure logic, shared between
        ├── defaults/        # server and future agent (worker) roles
        ├── handlers/
        ├── tasks/
        └── templates/
```

## Usage

```bash
cd ansible
cp inventory.ini.example inventory.ini   # fill in your own host IPs/users
ansible all -m ping                      # verify connectivity first
ansible-playbook playbook.yaml
```

The playbook installs a pinned RKE2 version, configures it as a server, waits for the node to report `Ready`, and pulls a working kubeconfig back to the control machine (`ansible/fetched/`, gitignored — contains live cluster credentials).

## Status

- [x] Single-node RKE2 control plane, bootstrapped via Ansible
- [ ] Additional nodes joining as agents (worker role)
- [ ] Persistent storage via `democratic-csi` (NFS backend on TrueNAS)
- [ ] GitOps deployment pipeline (Gitea + ArgoCD/Flux)
