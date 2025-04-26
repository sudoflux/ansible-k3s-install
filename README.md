# ansible-k3s-install

This project provides an Ansible playbook and inventory to:

- Wipe existing k3s and Flux installations from your cluster nodes
- Reinstall the latest k3s (minimal install, ready for Cilium CNI)
- Join worker nodes to the cluster
- Install the latest Flux on the master node

## Inventory

Edit the `hosts` file to match your cluster's DNS names or IPs. Example:

```
[master]
k3s-master1

[workers]
k3s1
k3s2
k3s3
```

## Usage

1. Ensure SSH key access is set up for all nodes.
2. Run the playbook:

```
ansible-playbook -i hosts wipe_and_reinstall_k3s_flux.yml
```

This will:
- Remove k3s and Flux from all nodes
- Install a minimal k3s (no flannel, traefik, or network policy)
- Join workers to the cluster
- Install Flux on the master

## Next Steps

After running the playbook, install your preferred CNI (e.g., Cilium) on the cluster.

---

Feel free to modify the playbook for your specific needs.
