# Test Environment

This folder contains a very small standalone Splunk lab environment intended as a warm-up before building larger distributed and clustered Splunk architectures.

The purpose of this environment is to:

- become familiar with Vagrant
- understand the provisioning workflow
- understand the Splunk installation script
- learn how automated infrastructure provisioning works
- verify that our Splunk installation script functions correctly
- create a simple disposable Splunk lab for experimentation

This environment provisions a single Ubuntu 22.04 virtual machine using Vagrant and automatically installs Splunk Enterprise through a shell provisioning script.

---

# Environment Overview

The environment contains:

| Component | Description |
|---|---|
| Ubuntu 22.04 VM | Base Linux operating system |
| VMware Desktop / VMware Fusion | Hypervisor provider |
| Vagrant | Infrastructure provisioning/orchestration |
| Splunk Enterprise | Automatically installed during provisioning |
| auditd | Linux audit subsystem used for security event generation |

---

# Hypervisor / Provider Notes

This lab was primarily developed and tested using the VMware Desktop provider:

- VMware Workstation (Windows)
- VMware Fusion (macOS)

However, Vagrant supports multiple providers. In principle, any supported Vagrant provider may be used, including:

- VMware Desktop
- VirtualBox
- Hyper-V
- libvirt
- Parallels

Users may choose whichever provider they are most comfortable with.

## VMware Workstation Version Recommendation

On Windows systems, VMware Workstation 17 is currently recommended.

During testing, VMware Workstation 24 showed issues with the installation and operation of the Vagrant VMware utility.

If problems occur with VMware Workstation 24, consider using VMware Workstation 17 instead.

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

Verify installed Vagrant plugins:

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

Before provisioning, inspect the current Vagrant state:

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

# auditd Demonstration and Linux Audit Log Onboarding

This standalone environment is also used to demonstrate onboarding Linux audit logs into Splunk.

The Linux audit subsystem (`auditd`) generates detailed security-relevant operating system events, including:

- file access
- permission changes
- authentication activity
- process execution
- privilege escalation
- system configuration changes

These events are written into:

```text
/var/log/audit/audit.log
```

Because audit logs are security-sensitive, they are typically only accessible by `root`.

Since Splunk runs as the dedicated Linux user `splunk`, we must grant controlled read access to the audit logs.

---

# Installing auditd

Install the audit subsystem:

```bash
apt-get update
apt-get install -y auditd audispd-plugins
```

Verify auditd is running:

```bash
systemctl status auditd
```

---

# Inspect Existing Audit Log Permissions

Inspect the audit log:

```bash
ls -l /var/log/audit/audit.log
```

Inspect existing ACLs:

```bash
getfacl /var/log/audit/audit.log
```

---

# Production-Style Audit Log Access Model

A common initial approach is using ACLs (`setfacl`) directly on the audit log file.

However, this often creates problems during log rotation because newly rotated log files may lose their ACL entries.

Instead, this lab demonstrates a cleaner and more production-oriented solution using:

- a dedicated Linux group
- auditd log group assignment
- controlled group membership

This avoids repeatedly managing ACLs on rotating audit log files.

---

# Create Dedicated Audit Reader Group

Create a dedicated group for systems allowed to read audit logs:

```bash
groupadd --system auditreaders
```

Add the Splunk Linux user to the group:

```bash
usermod -aG auditreaders splunk
```

Verify group membership:

```bash
sudo -u splunk groups
```

---

# Configure auditd Log Group

Edit the auditd configuration:

```bash
vim /etc/audit/auditd.conf
```

Locate the following parameter:

```ini
log_group =
```

Set it to:

```ini
log_group = auditreaders
```

This instructs auditd to assign the audit log group ownership to `auditreaders`.

---

# Restart Services

Restart auditd:

```bash
systemctl restart auditd
```

Restart Splunk:

```bash
systemctl restart Splunkd.service
```

---

# Verify Permissions

Inspect the audit directory:

```bash
ls -ld /var/log/audit
```

Inspect the audit log:

```bash
ls -l /var/log/audit/audit.log
```

The audit log should now show group ownership similar to:

```text
root auditreaders
```

---

# Verify Splunk User Access

Verify the Splunk Linux user can read the audit log:

```bash
sudo -u splunk head /var/log/audit/audit.log
```

If the file contents are displayed successfully, Splunk now has persistent read access to the Linux audit logs.

---

# Splunk Add-on for Unix and Linux

To correctly parse and normalize Linux audit events, install:

```text
Splunk Add-on for Unix and Linux
```

through the Splunk Web UI.

The add-on provides:
- Linux sourcetypes
- field extractions
- CIM mappings
- Linux data normalization

---

# Adding the Audit Log in Splunk Web

Inside Splunk Web:

```text
Settings → Add Data → Monitor
```

Select:

```text
Files & Directories
```

Monitor:

```text
/var/log/audit/audit.log
```

Recommended sourcetype:

```text
linux:audit
```

At this point, Linux audit events should begin appearing inside Splunk.

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

The goal is first to understand the fundamentals of automated infrastructure provisioning, Linux telemetry onboarding, and Splunk installation before introducing additional architectural complexity.