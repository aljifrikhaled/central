# Frappe Cloud: Beginner's Blueprint for AWS and Local Deployment

Welcome! This guide is designed to help you, even without a heavy technical background, understand how to set up Frappe Cloud. We will cover how to run it locally on your computer (using standard Linux tools) and what you need to look for when moving to AWS for production.

---

## 1. Understanding the Architecture (The "Four Systems")

Think of Frappe Cloud not as a single website you just turn on, but as a busy airport with different departments handling different tasks:

*   **Central (The Control Tower):** This is the main dashboard you and your users log into. It handles billing, creates teams, and lets you click "Create Server". (This is the code in this repository).
*   **Atlas (The Runway & Traffic Control):** Atlas manages the actual physical infrastructure. When you ask Central for a new server, Atlas is the system that finds a physical host, boots up a secure virtual machine (**Firecracker microVM**), and routes internet traffic to it.
*   **Cargo (The Airplane Factory):** Atlas doesn't build the servers from scratch. It asks **Cargo** to build a standard template (an image) with Ubuntu and Frappe pre-installed. Atlas then boots *that* image. Cargo is created and managed *by* Atlas.
*   **Pilot (The Flight Crew):** Pilot is a small manager that lives *inside* the virtual machine that Atlas booted. It manages the specific Frappe sites and databases inside that server.

---

## 2. Setting Up Locally for Development

If you want to test and develop the **Central** Control Tower dashboard on your computer, you don't need AWS yet. You can run it right on your local machine using standard Linux tools.

### Prerequisites (What you need installed first)
Before you run the commands below, make sure your system has these installed:
*   **Python** (version 3.14)
*   **Node.js** (version 24)
*   **Yarn**
*   **Pilot** CLI (Frappe's tool for managing benches)

### Step-by-Step Local Commands

Open your terminal and run these commands one by one to download the code, set up a local site, and start the system:

```bash
# 1. Download the Central codebase
pilot get-app https://github.com/frappe/central.git --branch develop

# 2. Create a new local site for Central
pilot new-site central.localhost

# 3. Install the Central app onto your new site
pilot install-app central.localhost central

# 4. Turn on developer mode so you can make changes
pilot frappe --site central.localhost set-config developer_mode 1

# 5. Start the backend server
pilot start
```

Your backend is now running!
*   You can log into the classic Frappe Desk interface at: `http://central.localhost:<port>/app`

**Starting the Modern Dashboard (Frontend)**
Central has a modern Vue.js dashboard. To see it, open a **second, separate terminal window**, and run:

```bash
# Navigate to the dashboard folder
cd apps/central/dashboard

# Install the frontend packages
yarn install

# Start the frontend dashboard
yarn dev
```

*   You can now view the modern Vue console at: `http://central.localhost:<port>/dashboard`

---

## 3. Moving to AWS Production: What You Need

When you are ready to put this on the internet for real users, a single server is not enough. You must deploy at least **2 distinct systems** because the Control Tower (Central) and the Runway (Atlas) must be separated.

Here is your AWS shopping list and what to search for:

### System 1: Central (The Control Plane)
*   **What it does:** Runs the Vue dashboard and billing APIs.
*   **AWS Search:** Standard **EC2 instances**. You can use normal compute instances for this, as it just runs standard web application code.

### System 2: Atlas (The Infrastructure Orchestrator)
*   **What it does:** Runs physical hosts that boot Firecracker microVMs to securely isolate your users' workloads.
*   **AWS Search:** You must search for **EC2 instances**. (When configuring AWS, ensure you select EC2 instance types that can support the virtualization requirements for the workloads).
*   *(Note: Atlas will automatically handle creating Cargo to build the images).*

### Important Note on Production Complexity
Frappe Cloud is an enterprise-scale tool. While the local setup is straightforward, configuring Atlas to talk to AWS, setting up IPv6 private networks, and routing wildcard domains (`*.yourcloud.com`) requires advanced DevOps knowledge. There is no simple "one-click" command for the AWS deployment in this repository.
