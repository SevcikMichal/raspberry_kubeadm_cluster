# Kubernetes Cluster Setup with Ansible

## Overview

This project was created to **automate the setup of a HomeLab Kubernetes cluster using Ansible**. Initially, the process was manual and involved running several individual scripts for the control plane and worker nodes. With this project, the entire setup is now automated, making the cluster setup process faster, more reliable, and easier to manage.

The goal of this project was to learn how Ansible works and apply it to a real-world use case, automating what was once a tedious manual setup for Kubernetes clusters.

## Features

- Automates the setup of both **control plane** and **worker nodes**.
- Applies **common configuration** to all nodes (such as disabling swap, installing container runtimes).
- Installs and configures Kubernetes components and dependencies.
- Automatically **joins worker nodes** to the Kubernetes cluster.
- Installs flux using bootstrap with private repo. Afterwards the management of the cluster is purely done via gitops.
- Modular structure with **Ansible roles** for common tasks, control plane setup, and worker node setup.

## Project Structure

```
raspberry_kubeadm_cluster/
├── ansible.cfg              # Ansible configuration file
├── inventory/
│   └── hosts                # Inventory file defining controlplane and workernodes
├── playbooks/
│   └── cluster_setup.yaml   # Main playbook that applies roles to controlplane and workernodes
└── roles/
    ├── common/              # Role for common tasks across all nodes
    ├── controlplane/        # Role for control plane specific tasks
    ├── flux/                # Role for setting up flux on newly created cluster
    ├── validate/            # Role for validating hosts
    └── workernodes/         # Role for worker nodes specific tasks
```

### `ansible.cfg`
This is the configuration file for Ansible, which defines settings like the inventory location, user access, and the path for roles.

### `inventory/hosts`
The inventory file specifies which nodes belong to the **controlplane** and **workernodes** groups. Example:

```ini
[controlplane]
192.168.1.10

[workernodes]
192.168.1.11
192.168.1.12
```

### `inventory/fluxbootstrap`
The inventory file specifies variables for bootstrapin flux. Use it to override default values in `roles/flux/vars/main.yaml`.

### `playbooks/cluster_setup.yaml`
This is the main playbook that runs the roles for both control plane and worker nodes.

### Roles
- **common**: Tasks that are applied to both control plane and worker nodes (e.g., disabling swap, setting up container runtime).
- **controlplane**: Tasks specific to the control plane, such as initializing Kubernetes and configuring the master node.
- **flux**: Tasks specific to setting up flux. These are run on the ansible control plane to test out connection to the cluster.
- **validate**: Tasks specific to validate hosts. There are some assumptions in the later steps that would not work if the expectations are not met.
- **workernodes**: Tasks specific to worker nodes, such as joining them to the Kubernetes cluster.

## Prerequisites

- **Ansible** must be installed on your local machine.
- Raspberries should already have os running (Tested on Rasberry OS Lite). 
- Ensure you can SSH into all nodes (control plane and worker nodes) using key-based authentication.

## Usage

1. **Clone this repository**:
   ```bash
   git clone <repository-url>
   cd ansible-cluster-setup
   ```

2. **Configure the inventory**:
   Update the `inventory/hosts` file with the IP addresses of your control plane and worker nodes.

3. **Update the `ansible.cfg`** (optional):
   Modify the `ansible.cfg` file if you need to change SSH settings or other Ansible defaults.

4. **Test the connection**:
   Before running the playbook, ensure Ansible can reach all the nodes by running:
   ```bash
   ansible all -m ping
   ```

5. **Run the playbook**:
   To execute the playbook and set up your cluster, run:
   ```bash
   ansible-playbook playbooks/cluster_setup.yaml
   ```

6. **Verify**:
   Once the playbook has run, log into your control plane node and check if the worker nodes have successfully joined the cluster:
   ```bash
   kubectl get nodes
   ```
   Kubeconfig should be available at `raspberry_kubeadm_cluster/out` after running the playbook successfully.

## How It Works

- **Common Role**: The `common` role applies to both control plane and worker nodes. It installs necessary dependencies, disables swap, and sets up the container runtime.
  
- **Control Plane Role**: The `controlplane` role handles the initialization of the Kubernetes control plane. It runs `kubeadm init`, installs Flannel as the network plugin, and prepares the control plane for worker nodes to join.

- **Flux Role**: The `flux` role takes care of installing flux on the newly created cluster. It is run on the Ansible controlplane to test out the connection to the cluster from outside. This role is also installing the flux cli onto the system. Therefore sudo password is required use `--ask-become-pass` to input it. This is currently intended by design.

- **Worker Node Role**: The `workernodes` role pulls the required Kubernetes images and runs `kubeadm join` to connect the worker nodes to the control plane.

## Why This Project?

Before this project, I manually executed scripts to set up each part of the Kubernetes cluster. This was time-consuming and error-prone, especially when scaling the cluster or repeating the process. With Ansible, I learned how to automate the entire process, allowing me to set up the cluster much faster and more efficiently.

Now, the setup is completely automated, reducing the manual steps and ensuring a consistent and repeatable setup across multiple environments.

## Future Improvements

- Add more roles for monitoring, storage, and logging setup.
- Automate additional configurations like high availability for the control plane.
- Implement advanced features like deploying applications via Ansible.
- Get rid of the sudo requirement on runner.