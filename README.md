Okay, here is the README content formatted as raw markdown text. You should be able to copy and paste this directly into your `README.md` file.

---

# Ansible Playbook for Minimal k3s Cluster Setup (for Flux/Cilium)

This Ansible playbook automates the process of setting up a fresh, minimal Kubernetes cluster using k3s. It is specifically designed to prepare nodes for a GitOps workflow managed by Flux, where Flux itself will be responsible for installing the CNI (Container Network Interface), such as Cilium.

The playbook performs the following actions:

1.  **Wipes** existing k3s and Flux installations from all target nodes.
2.  **Reboots** the nodes reliably after the wipe using the OS scheduler (`shutdown -r +1`).
3.  **Waits** for nodes to come back online after reboot.
4.  **Installs** a minimal k3s server on the designated master node, disabling Flannel CNI, Traefik, and ServiceLB.
5.  **Installs** the k3s agent on designated worker nodes, connecting them to the master.
6.  **Installs** the Flux CLI tool (`flux`) on the master node.
7.  **Sets up** `kubeconfig` for the specified user on the master node for immediate `kubectl` and `flux` access.

The key outcome is a k3s cluster where **all nodes are intentionally in a `NotReady` state**, awaiting the CNI installation managed via Flux.

## Prerequisites

1.  **Ansible:** Ansible must be installed on the machine where you will run the playbook.
2.  **SSH Access:** You need SSH access (preferably key-based) from your control machine to all target nodes (master and workers).
3.  **Sudo Privileges:** The SSH user needs `sudo` privileges on the target nodes (ideally configured for passwordless `sudo` for Ansible's `become`).
4.  **Python:** Target nodes need Python installed for Ansible modules to function (usually Python 2.7 or Python 3.x). Most modern Linux distributions have this.
5.  **jq:** The `jq` package is used by the wipe script for robust Flux finalizer removal. The playbook attempts to install it, but pre-installing it on nodes is recommended.
6.  **Inventory File:** An Ansible inventory file defining your master and worker nodes.

## Setup

1.  **Clone the Repository:**

    `git clone <your-repo-url>`
    `cd <your-repo-directory>`

2.  **Configure Inventory:** Create or modify an Ansible inventory file (e.g., `inventory.yaml` or `hosts`). It **must** contain:
    *   A `[master]` group with **exactly one** host defined.
    *   A `[workers]` group with one or more worker hosts.

    **Example (`inventory.yaml`):**

    ```
    all:
      vars:
        ansible_user: josh # Replace with your SSH user
        # ansible_ssh_private_key_file: /path/to/your/ssh/key # Optional: if needed
      children:
        master:
          hosts:
            k3s-master1: # This hostname MUST match the var in Play 4
              ansible_host: 192.168.1.100 # Replace with master IP/hostname
        workers:
          hosts:
            k3s1:
              ansible_host: 192.168.1.101 # Replace with worker 1 IP/hostname
            k3s2:
              ansible_host: 192.168.1.102 # Replace with worker 2 IP/hostname
            k3s3:
              ansible_host: 192.168.1.103 # Replace with worker 3 IP/hostname

    ```

3.  **Verify Master Hostname Variable:** Open the main playbook file (`rebuild_k3s_flux.yaml` or similar) and locate **Play 4: Install k3s Agent on Workers**. Ensure the `k3s_master_hostname` variable matches the hostname you defined for your master in the inventory file (e.g., `k3s-master1` in the example above).

    ```
    # In Play 4:
    vars:
      k3s_master_hostname: "k3s-master1" # <-- MAKE SURE THIS MATCHES YOUR INVENTORY MASTER HOSTNAME!
      # ...

    ```

4.  **(Optional) Verify Target User:** In **Play 5: Install Flux CLI and Setup Kubeconfig**, the `target_user` and `target_user_home` variables default to using `ansible_user`. If you need the kubeconfig set up for a different user on the master, adjust these variables.

## Usage

Run the playbook from your control machine, pointing it to your inventory file:

`ansible-playbook -i inventory.yaml rebuild_k3s_flux.yaml`

*(Replace `inventory.yaml` and `rebuild_k3s_flux.yaml` with your actual file names if different.)*

The playbook will execute the wipe, reboot, install, and setup steps across all defined nodes.

## Expected Outcome

After the playbook completes successfully:

*   All nodes (master and workers) will be listed by `kubectl get nodes` (run on the master).
*   **All nodes will show `STATUS` as `NotReady`**. This is the **correct and expected** state, as no CNI has been installed yet.
*   The `flux` CLI will be available at `/usr/local/bin/flux` on the master node.
*   A functional `kubeconfig` file will be located at `~/.kube/config` (relative to the `target_user`) on the master node.

## Next Steps: Flux Bootstrap

The cluster is now ready for Flux to take over. Log in to your **master node** and run the `flux bootstrap` command relevant to your Git provider (e.g., GitHub, GitLab).

**Example for GitHub:**

```
# Run this ON the k3s-master1 node
flux bootstrap github \
  --owner=<your-github-username-or-org> \
  --repository=<your-gitops-repo-name> \
  --branch=main \
  --path=./clusters/my-cluster \ # Path within the repo Flux should manage
  --personal # Use if it's a personal repo, omit for org repo

```

Ensure your GitOps repository (at the specified `--path`) contains the Kubernetes manifests for Flux to install your desired CNI (e.g., Cilium). Once Flux applies the CNI manifests, the nodes should transition to the `Ready` state.

## Contributing

Contributions are welcome! Please feel free to open an issue or submit a pull request.

## License

[Specify your license here, e.g., MIT License]

---