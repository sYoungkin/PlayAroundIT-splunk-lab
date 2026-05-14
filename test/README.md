# Test Environment

This folder contains a very small standalone Splunk lab environment intended as a warm-up before building larger distributed and clustered Splunk architectures.

The purpose of this environment is to:

- become familiar with Vagrant
- understand the provisioning workflow
- understand the Splunk installation script
- learn how automated infrastructure provisioning works
- verify that our Splunk installation script functions correctly
- create a simple disposable Splunk lab for experimentation

This environment provisions a single Ubuntu 22.04 virtual machine using Vagrant and VMware Desktop and automatically installs Splunk Enterprise through a shell provisioning script.

---

# Environment Overview

The environment contains:

| Component | Description |
|---|---|
| Ubuntu 22.04 VM | Base Linux operating system |
| VMware Desktop | Hypervisor provider |
| Vagrant | Infrastructure provisioning/orchestration |
| Splunk Enterprise | Automatically installed during provisioning |

---

# Prerequisites

Before starting, ensure the following are installed:

- Vagrant
- VMware Desktop / VMware Fusion
- Vagrant VMware Desktop plugin

Verify Vagrant is installed:

```bash
vagrant --version
```

Verify VMware plugin:

```bash
vagrant plugin list
```

---

# Starting the Environment

Navigate into the test environment folder:

```bash
cd test
```

---

# Check Environment Status

Before provisioning, we can inspect the current Vagrant state:

```bash
vagrant status
```

Initially, the machine should show:

```text
test    not created
```

---

# Provision the Environment

Create and provision the virtual machine:

```bash
vagrant up
```

During provisioning, Vagrant will:

1. Download the Ubuntu base box (first run only)
2. Create the virtual machine
3. Boot the operating system
4. Execute the Splunk installation script
5. Install and configure Splunk Enterprise
6. Enable Splunk boot-start via systemd
7. Start Splunk automatically

Provisioning may take several minutes during the first run.

---

# Access Splunk Web

At the end of the provisioning process, the installation script automatically prints the hostname, IP address, and Splunk Web URL for the VM.

Example:

```text
[INFO] Installation complete.
[INFO] Hostname: splunk-test
[INFO] Splunk Web: http://192.168.188.128:8000
[INFO] Username: admin
[INFO] Password: ChangeMe123!
```

The VM receives its IP address dynamically through the VMware DHCP network. The installation script automatically detects the assigned IP address and prints the correct Splunk Web URL during provisioning.

Open the displayed URL in a browser:

```text
http://<IP_ADDRESS>:8000
```

Example:

```text
http://192.168.188.128:8000
```

---

# Login Credentials

Login with:

| Field | Value |
|---|---|
| Username | admin |
| Password | Defined in the Vagrantfile environment variable |

The password is passed into the installation script during provisioning through the Vagrant shell provisioner environment configuration.

---

# Verify Splunk Status

A system-wide alias named `splunk` is automatically configured during provisioning.

Verify Splunk is running:

```bash
splunk status
```

You should see output similar to:

```text
splunkd is running
splunk helpers are running
```

---

# Test Root Access

Switch to the root user:

```bash
sudo su -
```

Verify the alias also works for root:

```bash
splunk status
```

Because the alias is configured through `/etc/profile.d/`, it is available system-wide for all users.

---

# Destroying the Environment

Because the environment is fully automated, it can easily be destroyed and recreated.

Destroy the VM:

```bash
vagrant destroy -f
```

Recreate it:

```bash
vagrant up
```

This allows rapid experimentation without worrying about permanently damaging the system.

---

# Purpose of this Environment

This standalone environment serves as a preparation step before building more advanced Splunk architectures, including:

- distributed search
- deployment servers
- heavy forwarders
- indexer clustering
- search head clustering
- ingest pipelines
- Cribl integration
- TLS-enabled environments
- monitoring and observability tooling

The goal is first to understand the fundamentals of automated infrastructure provisioning and Splunk installation before introducing additional architectural complexity.
