# WIP:  Vagrant-Datalake: Data Lake Architecture

This repository provides a Vagrant-based setup for a scalable data lake architecture, enabling users to experiment with and deploy modern data storage, processing, and load-balancing solutions.  Options configurable by changing settings.yaml.

### Infrastructure Overview:
- **Total Servers**: 7
  - **Storage Nodes**: 2 MinIO servers configured as a storage cluster for high-performance object storage.
  - **Load Balancing Nodes**: 2 servers running NGINX and Keepalived for load balancing and high availability.
  - **Compute Node**: 2 servers (master and worker) equipped with Apache Spark, Delta Lake, Iceberg, and Hudi for advanced data processing and analysis.
  - **Automation Node**: 1 server with Apache Airflow

## Prerequisites

- [Vagrant](https://www.vagrantup.com/) installed (see [Installing Vagrant](#installing-vagrant) below).
- [VirtualBox](https://www.virtualbox.org/) (or another supported provider) installed as the virtualization backend.
- Git installed to clone the repository (see [Cloning the Repository](#cloning-the-repository)).

## Getting Started

1. Clone the repository (see [Cloning the Repository](#cloning-the-repository)).
2. Navigate to the repository directory:
   ```bash
   cd vagrant-data-engineering
   ```
3. Start the Vagrant environment:
   ```bash
   vagrant up
   ```
   This will provision five VMs and install all supporting applications.

4. Wait for the provisioning to complete (this may take a few minutes depending on your system and network).

## Cloning the Repository

To get the code from GitHub:

```bash
git clone https://github.com/rerichardjr/vagrant-data-engineering.git
cd vagrant-data-engineering
```

- **Requirements**: Git must be installed.
  - On Ubuntu: `sudo apt install git`
  - On macOS: `brew install git` (with Homebrew) or via Xcode tools.
  - On Windows: Download from [git-scm.com](https://git-scm.com/) or use `winget install --id Git.Git`.

## Installing Vagrant

### On Ubuntu/Debian
```bash
sudo apt update
sudo apt install vagrant
```

### On macOS
Using Homebrew:
```bash
brew install vagrant
```
Or download the installer from [vagrantup.com](https://www.vagrantup.com/downloads).

### On Windows
1. Download the installer from [vagrantup.com](https://www.vagrantup.com/downloads).
2. Run the installer and follow the prompts.
3. Alternatively, use Winget:
   ```bash
   winget install Hashicorp.Vagrant
   ```

### Verify Installation
```bash
vagrant --version
```

## Verifying the Cluster with Nodetool

To check the cluster status:

1. SSH into a VM:
   - Use `vagrant ssh` to connect to one of the nodes (e.g., `dl-compute1`):
     ```bash
     vagrant ssh dl-compute1
     ```
   - This connects you to the `dl-compute1` VM.


## Cleaning Up

To stop and remove the VMs:
```bash
vagrant destroy -f
```

---

## About My Lab

This repository is supported by a home lab designed for hands-on network and server experimentation.  Below are the details of the infrastructure:

### Server Configuration
- **Motherboard**: ASRock 990FX Extreme9  
- **Processor**: AMD FX™-8150 Eight-Core, 3.6 GHz  
- **Memory**: 32 GiB  
- **Storage**: 480 GB Kingston SSD  
- **Operating System**: Ubuntu 24.04, deployed via Canonical MAAS (PXE boot)  

### Networking Setup
- **NICs**:  
  - Intel Corporation 82571EB/82571GB Gigabit Ethernet Controller (copper applications)  
    - Interfaces: `enp6s0f0`, `enp6s0f1`  
  - Intel Corporation 82583V Gigabit Network Connection  
    - Interface: `enp9s0`  
- **Traffic Management**:  
  - Open vSwitch (OVS) bridge configured with `enp6s0f0` and `enp6s0f1` for lab traffic  
- **Network Integration**:  
  - VLANs for lab traffic are trunked to a MikroTik router for network segmentation and control
