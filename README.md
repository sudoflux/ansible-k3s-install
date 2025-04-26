# ansible-k3s-install

This project provides Ansible playbooks and inventory to:

- Wipe existing k3s and Flux installations from your cluster nodes
- Reboot nodes to ensure a clean state
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
2. Wipe and reboot all nodes:

```
ansible-playbook -i hosts k3s_wipe_and_reboot.yml
```

   - You will see `UNREACHABLE` errors for all hosts as they reboot. This is normal.
   - Wait for all nodes to finish rebooting and become reachable again.

3. Install k3s and Flux:

```
ansible-playbook -i hosts k3s_install_and_flux.yml
```

   - This will:
     - Install a minimal k3s (no flannel, traefik, or network policy)
     - Join workers to the cluster
     - Install Flux on the master

## Next Steps

After running the playbooks, install your preferred CNI (e.g., Cilium) on the cluster.

---

Feel free to modify the playbooks for your specific needs.
