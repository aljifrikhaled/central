# Frappe Cloud on AWS: Analysis and Deployment Guide

This document provides an analysis of the Frappe Cloud architecture, dependencies, and a detailed guide addressing the feasibility of running a production environment on AWS.

## Project Architecture & Repositories

Frappe Cloud v2 is an enterprise-scale orchestration platform, not a standard monolithic web application. It relies on four distributed systems that work in tandem:

1. **Central**: The control plane (this repository). It manages teams, permissions, billing, servers, and acts as the central user interface.
2. **Atlas**: The system that manages regional infrastructure. Atlas places servers on physical hosts, configures networks, manages web proxying, and boots virtual machines. Crucially, each server is a **Firecracker microVM**.
3. **Cargo**: The image builder and regional storage manager. It builds the system images (Ubuntu with Frappe/Pilot pre-installed) that Atlas uses to boot VMs.
4. **Pilot**: The bench manager running inside the deployed servers. It manages the Frappe sites, databases, and applications inside a single virtual machine.

## Dependencies

To run the local development setup (specifically for the Central control plane), the following dependencies are required:
- **Python 3.14**
- **Frappe v16**
- **Node.js 24**
- **Yarn**

## Interfaces

The system provides two primary interfaces:
- **Vue Console**: A custom Vue 3 dashboard (`/dashboard`) built with Frappe UI. This is the customer-facing control panel for managing teams, servers, and billing.
- **Frappe App (Desk)**: The standard Frappe Desk interface (`/app`) used for administrative and backend management.

## Managing a Production Environment on AWS

The request is for a complete, step-by-step guide with commands for a non-technical person to run and manage the cloud on an AWS production environment with ease.

**It is important to emphasize that Frappe Cloud v2 is a complex infrastructure platform.** Setting up a production environment is an advanced DevOps engineering task and is not designed to be easily deployed by non-technical users. There is no simple copy-paste guide for production.

Here are the specific challenges and AWS requirements:

### 1. Infrastructure & Firecracker Requirement
Because Atlas uses **Firecracker microVMs** to run individual servers, it requires virtualization capabilities at the hardware level (KVM). Standard AWS EC2 instances do not support this natively.
- **AWS Settings**: You must use **AWS EC2 Bare Metal instances** (e.g., `i3.metal`, `m5.metal`, or `mac` instances depending on the region) which are highly expensive. Alternatively, you would need to use instances that explicitly support nested virtualization.

### 2. Why a Single-Server Production Setup is Unviable
You cannot deploy Frappe Cloud v2 on a single server for production. The architecture strictly splits the control plane from the data/compute plane:
- **Separation of Concerns:** The **Central** system (this repository) is just the control plane (the console and billing). It does not run user applications.
- **Infrastructure Requirements:** The user workloads are orchestrated by **Atlas** running on separate, dedicated bare-metal regional hosts. Atlas creates secure isolation using Firecracker microVMs. It cannot be combined on the same single standard EC2 instance with the Central control plane.
- **Image Building:** A third separate system, **Cargo**, builds the Ubuntu/Frappe images.
- **No Single-Node Installer:** Unlike older monolithic platforms, Frappe Cloud v2 has no automated "single-node" setup script for production. It is explicitly designed as a multi-node distributed orchestrator for a commercial cloud offering.

### 3. Advanced Networking
Atlas manages its own network stack:
- It configures an IPv6 private network for the Firecracker VMs.
- It requires a regional proxy for handling wildcard domains (e.g., `*.bom.yourcloud.com`).
- It uses WireGuard gateways for server access.

### 4. Lack of Official Production Tooling
The official repository documentation explicitly provides instructions for a **local development setup** (using the `pilot` CLI). It does not contain a single script, Terraform module, or guide for automated production cloud deployments. Frappe uses this system internally for their commercial offering, but the orchestration and bare-metal provisioning scripts are not part of a simple "start" script.

## Local Development Setup

If you want to run the **Central console** locally to explore it (not for production), you can follow the official local setup guide:

1. Install Pilot, create, and initialize a development bench.
2. Run these commands from your bench directory:
   ```bash
   pilot get-app https://github.com/frappe/central.git --branch develop
   pilot new-site central.localhost
   pilot install-app central.localhost central
   pilot frappe --site central.localhost set-config developer_mode 1
   pilot start
   ```
3. Open `http://central.localhost:<http_port>/dashboard` for the Vue console or `/app` for the Frappe Desk.
4. For the frontend development server, run in another terminal:
   ```bash
   cd apps/central/dashboard
   yarn install
   yarn dev
   ```

## Conclusion

Running Frappe Cloud on AWS for production is a complex infrastructure engineering task requiring deep knowledge of Linux, Firecracker/KVM virtualization, IPv6 networking, and distributed systems. A non-technical person cannot spin this up easily. If you require managed Frappe hosting, using the official Frappe Cloud service or a simpler standard Frappe/ERPNext Docker deployment is highly recommended instead of attempting to run the entire cloud orchestrator yourself.
