# Linux Server & Active Directory Administration Lab

**Ubuntu Server 24.04.4 LTS | Samba Active Directory | DNS | Kerberos | Identity Management | SMB File Sharing**

## Project Overview

This project documents the deployment, administration, and troubleshooting of a Linux-based Active Directory environment using Ubuntu Server and Samba.

The lab was designed to simulate real-world IT Support, Help Desk, and Junior System Administrator responsibilities, including user provisioning, password resets, authentication troubleshooting, security group administration, and shared-folder access control.

Rather than simply installing a server, I performed administrative tasks, investigated configuration problems, tested authentication, and verified security restrictions using authorized and unauthorized domain accounts.

### Project Objectives

- Configure an Ubuntu Server virtual machine.
- Administer a Samba Active Directory Domain Controller.
- Investigate DNS configuration and domain service discovery issues.
- Verify Kerberos authentication.
- Create and manage Active Directory users and security groups.
- Perform account disabling, re-enabling, and password resets.
- Configure an SMB departmental share.
- Test group-based access restrictions and least-privilege controls.
- Document troubleshooting procedures and verification results.

---

## Lab Environment

| Component | Configuration |
|---|---|
| Host Machine | Apple Silicon Mac (M4) |
| Virtualization Platform | VMware Fusion |
| Guest Operating System | Ubuntu Server 24.04.4 LTS (ARM64) |
| Directory Service | Samba Active Directory Domain Controller |
| Samba Version | 4.19.5 |
| Server Hostname | dc01 |
| Active Directory Domain | nishatlab.local |
| Kerberos Realm | NISHATLAB.LOCAL |
| NetBIOS Domain | NISHATLAB |
| Network Configuration | VMware NAT |
| Server IP Address | 192.168.204.130 |
| Internal DNS | Samba Internal DNS |
| Authentication Protocol | Kerberos |
| File Sharing Protocol | SMB/CIFS |

### Tools & Technologies

- Ubuntu Server and Linux CLI
- VMware Fusion
- Samba Active Directory
- `samba-tool`
- `testparm`
- Kerberos (`kinit`, `klist`)
- DNS utilities (`host`, `resolvectl`)
- Netplan
- `systemctl` and `journalctl`
- `smbclient`
- Linux filesystem permissions

## Case Study Summary

| Case | Technical Focus | Result |
|---|---|---|
| 01 | Ubuntu Server & Active Directory Environment | Server environment established |
| 02 | DNS Configuration & Troubleshooting | Internal AD DNS verified; external forwarding issues investigated |
| 03 | Kerberos Authentication | Domain authentication verified |
| 04 | User & Security Group Management | Domain users and groups created |
| 05 | User Account Lifecycle Management | Account administration and authentication verified |
| 06 | SMB File Sharing & Access Control | Authorized access succeeded; unauthorized access denied |

---

## Case 01 — Ubuntu Server & Active Directory Environment

### Objective

Establish a virtualized Ubuntu Server environment capable of providing centralized Active Directory services.

### Environment Setup

1. Installed Ubuntu Server 24.04.4 LTS (ARM64) using VMware Fusion.
2. Configured the virtual machine to use VMware NAT networking.
3. Established the server hostname `dc01`.
4. Configured Samba Active Directory Domain Controller services.
5. Established the Active Directory domain `nishatlab.local`.
6. Configured the Kerberos realm `NISHATLAB.LOCAL`.
7. Verified the operating system and server configuration.

### Verification

The following commands can be used to inspect the server environment:

```bash
hostnamectl
```

```bash
sudo systemctl status samba-ad-dc
```

```bash
sudo testparm -s
```

### Screenshot Evidence

**Screenshot 01 — Ubuntu Server Version**

![Ubuntu Server Version](Screenshots/01-ubuntu-server-version.png)

### Outcome

Established an Ubuntu Server virtual machine for Active Directory administration, domain authentication, DNS troubleshooting, and SMB file sharing.

The environment provided the foundation for the remaining five case studies.

**Skills Demonstrated:** Linux Administration, Virtualization, Server Configuration, Active Directory Fundamentals.

---

## Case 02 — Active Directory DNS Configuration & Troubleshooting

### Problem

During the Samba Active Directory configuration, DNS resolution issues affected domain service discovery.

Because Active Directory relies on DNS to locate domain controllers and authentication services, resolving these issues was important for reliable domain operations.

The investigation covered:

- Samba DNS service errors
- DNS resolver configuration
- Port 53 conflicts
- LDAP and Kerberos DNS records
- Netplan DNS settings
- Active Directory DNS zone records
- External DNS forwarding

### Troubleshooting Steps

#### Step 1 — Investigated Samba DNS Errors

Reviewed Samba service logs to identify DNS-related errors and possible service configuration problems.

```bash
sudo journalctl -u samba-ad-dc --no-pager
```

**Screenshot 02 — Samba DNS Error Logs**

![Samba DNS Error Logs](Screenshots/02-samba-dns-error-logs.png)

#### Step 2 — Examined DNS Resolver Settings

Inspected the server's DNS resolver configuration to identify which DNS servers were being used.

```bash
resolvectl status
```

Reviewed the relationship between the operating system's resolver settings and Samba's internal DNS service.

**Screenshot 03 — Initial DNS Server Settings**

![Initial DNS Server Settings](Screenshots/03-initial-dns-server-settings.png)

#### Step 3 — Investigated Port 53 Conflicts

Checked which services were listening on DNS port 53.

```bash
sudo ss -tulpn | grep ':53'
```

Investigated possible conflicts involving Samba DNS and `systemd-resolved`.

**Screenshot 04 — DNS Port 53 Conflict**

![DNS Port 53 Conflict](Screenshots/04-dns-port-53-conflict.png)

#### Step 4 — Verified Active Directory LDAP Records

Tested whether the DNS service could resolve the LDAP SRV record used for domain controller discovery.

```bash
host -t SRV _ldap._tcp.nishatlab.local ::1
```

**Screenshot 05 — Active Directory LDAP DNS Record**

![Active Directory LDAP DNS Record](Screenshots/05-active-directory-ldap-dns-record.png)

#### Step 5 — Verified Kerberos DNS Records

Tested the Kerberos service discovery record.

```bash
host -t SRV _kerberos._udp.nishatlab.local ::1
```

**Screenshot 06 — Kerberos DNS Record**

![Kerberos DNS Record](Screenshots/06-kerberos-dns-record.png)

#### Step 6 — Retested Internal DNS Resolution

After investigating the DNS configuration, repeated internal DNS queries to verify domain controller and Kerberos record resolution.

```bash
host -t A dc01.nishatlab.local ::1
```

```bash
host -t SRV _kerberos._udp.nishatlab.local ::1
```

**Screenshot 07 — Kerberos DNS Resolution Verification**

![Kerberos DNS Resolution Verification](Screenshots/07-kerberos-dns-resolution-fixed.png)

#### Step 7 — Reviewed Netplan Configuration

Inspected and adjusted the Ubuntu network configuration to address DNS resolver behavior.

Validated the Netplan configuration using:

```bash
sudo netplan generate
```

**Screenshot 08 — Netplan DNS Configuration**

![Netplan DNS Configuration](Screenshots/08-netplan-dns-configuration.png)

#### Step 8 — Investigated External DNS Forwarding

Tested external DNS resolution through Samba and the upstream DNS server.

```bash
host -t A google.com ::1
```

```bash
host -t A google.com 192.168.204.2
```

These tests helped distinguish problems with Samba's local forwarding behavior from upstream DNS resolution.

**Screenshot 09 — Samba DNS Forwarding Test**

![Samba DNS Forwarding Test](Screenshots/09-samba-dns-forwarding-success.png)

External DNS forwarding through Samba remained inconsistent during testing. Successful upstream DNS resolution did not establish that Samba forwarding was consistently working.

#### Step 9 — Verified Active Directory DNS Zone Records

Queried the Active Directory DNS zone using Samba's administration tool.

```bash
samba-tool dns query dc01.nishatlab.local nishatlab.local @ ALL -U Administrator
```

The DNS zone query displayed records including:

- SOA record
- NS record
- Domain Controller A record
- Active Directory DNS subdomains

**Screenshot 22 — Active Directory DNS Zone Verification**

![Active Directory DNS Zone Verification](Screenshots/22-active-directory-dns-zone-verification.png)

This provided additional evidence that the internal Active Directory DNS zone was configured and accessible.

#### Step 10 — Verified Samba DNS Forwarder Configuration

Inspected the DNS forwarder configured in Samba.

```bash
sudo samba-tool testparm --parameter-name="dns forwarder"
```

The configuration returned:

```text
192.168.204.2
```

**Screenshot 23 — Samba DNS Forwarder Configuration**

![Samba DNS Forwarder Configuration](Screenshots/23-samba-dns-forwarder-configuration.png)

This confirmed the configured upstream DNS forwarder. However, verifying the configuration value alone did not confirm successful external DNS forwarding.

### Root Cause / Findings

The investigation identified DNS resolver configuration issues and potential port 53 conflicts that required troubleshooting.

Internal Active Directory DNS records were successfully verified after configuration adjustments.

External DNS forwarding remained a separate issue, with some queries through Samba continuing to fail despite successful direct queries to upstream DNS servers.

### Verification Results

| Test | Result |
|---|---|
| LDAP SRV record lookup | Verified |
| Kerberos SRV record lookup | Verified |
| Domain Controller A record lookup | Verified |
| Active Directory DNS zone query | Successful |
| Samba DNS forwarder configuration | Verified |
| External DNS forwarding through Samba | Inconsistent / unresolved |

### Outcome

Successfully verified internal Active Directory DNS functionality and service discovery records.

Gained practical experience diagnosing Linux DNS configuration problems through log analysis, resolver inspection, network configuration, DNS queries, and Samba administration tools.

**Skills Demonstrated:** DNS Troubleshooting, Linux Networking, Active Directory DNS, Service Discovery, Netplan, Log Analysis.

---

## Case 03 — Kerberos Authentication

### Objective

Verify that the Samba Active Directory environment supports domain authentication using Kerberos.

### Steps Performed

#### Step 1 — Requested a Kerberos Ticket

Authenticated using the domain Administrator account.

```bash
kinit Administrator@NISHATLAB.LOCAL
```

Kerberos was used to request a Ticket Granting Ticket (TGT) from the domain's authentication service.

#### Step 2 — Verified Authentication

Inspected the Kerberos ticket cache.

```bash
klist
```

**Screenshot 10 — Kerberos Authentication Success**

![Kerberos Authentication Success](Screenshots/10-kerberos-authentication-success.png)

#### Step 3 — Examined Kerberos Ticket Information

Reviewed the ticket information, including the authenticated principal and ticket-granting service.

```bash
klist
```

**Screenshot 11 — Kerberos Authentication Verification**

![Kerberos Authentication Verification](Screenshots/11-kerberos-authentication-verification.png)

### Verification Results

| Test | Result |
|---|---|
| Administrator Kerberos authentication | Successful |
| Kerberos ticket request | Successful |
| Ticket cache inspection | Verified |

### Outcome

Successfully authenticated a domain account using Kerberos and verified the resulting authentication ticket.

This demonstrated the relationship between Active Directory, DNS service discovery, and Kerberos authentication.

**Skills Demonstrated:** Kerberos, Domain Authentication, Identity Verification, Authentication Troubleshooting.

---

## Case 04 — Active Directory User & Security Group Management

### Objective

Simulate routine IT Support responsibilities involving Active Directory user provisioning and security group administration.

### Steps Performed

#### Step 1 — Created a Domain User

Created a standard domain user named `sarah`.

```bash
sudo samba-tool user create sarah
```

Verified the account details.

```bash
sudo samba-tool user show sarah
```

**Screenshot 12 — Active Directory User Creation**

![Active Directory User Creation](Screenshots/12-active-directory-user-creation.png)

#### Step 2 — Created a Security Group

Created an Active Directory security group named `IT-Support`.

```bash
sudo samba-tool group add IT-Support
```

**Screenshot 13 — Active Directory Security Group**

![Active Directory Security Group](Screenshots/13-active-directory-security-group.png)

#### Step 3 — Assigned Group Membership

Added Sarah to the IT-Support security group.

```bash
sudo samba-tool group addmembers IT-Support sarah
```

#### Step 4 — Verified Group Membership

Listed the group's members.

```bash
sudo samba-tool group listmembers IT-Support
```

**Screenshot 14 — Security Group Membership**

![Security Group Membership](Screenshots/14-security-group-membership.png)

### Verification Results

| Test | Result |
|---|---|
| Domain user creation | Successful |
| User account lookup | Successful |
| Security group creation | Successful |
| Group membership assignment | Successful |
| Group membership verification | Successful |

### Outcome

Successfully created an Active Directory user, configured a security group, and verified group membership.

The IT-Support security group was later used to enforce access restrictions on an SMB departmental share.

**Skills Demonstrated:** Active Directory Administration, User Provisioning, Security Groups, IAM, Group Membership Management.

---

## Case 05 — Active Directory User Account Lifecycle Management

### Objective

Simulate common Help Desk account administration tasks involving account disabling, re-enabling, password resets, and authentication verification.

### Scenario

A domain user's account requires administrative changes during its lifecycle.

These actions represent routine identity management responsibilities performed by IT Support teams.

### Steps Performed

#### Step 1 — Disabled the User Account

Disabled Sarah's Active Directory account.

```bash
sudo samba-tool user disable sarah
```

Inspected the account status to verify the change.

**Screenshot 15 — User Account Disabled**

![User Account Disabled](Screenshots/15-user-account-disabled.png)

#### Step 2 — Re-enabled the User Account

Restored the account to an enabled state.

```bash
sudo samba-tool user enable sarah
```

Verified the updated account status.

**Screenshot 16 — User Account Re-enabled**

![User Account Re-enabled](Screenshots/16-user-account-reenabled.png)

#### Step 3 — Reset the User Password

Performed an administrative password reset.

```bash
sudo samba-tool user setpassword sarah
```

**Screenshot 17 — User Password Reset**

![User Password Reset](Screenshots/17-user-password-reset.png)

#### Step 4 — Verified Authentication

Tested authentication using Sarah's updated credentials.

```bash
kinit sarah@NISHATLAB.LOCAL
```

Checked the resulting Kerberos ticket.

```bash
klist
```

**Screenshot 18 — User Kerberos Authentication**

![User Kerberos Authentication](Screenshots/18-user-kerberos-authentication.png)

### Verification Results

| Administrative Task | Result |
|---|---|
| Disable domain user | Successful |
| Re-enable domain user | Successful |
| Reset user password | Successful |
| Authenticate with updated credentials | Successful |
| Verify Kerberos ticket | Successful |

### Outcome

Successfully demonstrated account lifecycle management, including disabling and restoring a domain account, resetting its password, and verifying authentication.

These procedures reflect common IT Support and Help Desk account management responsibilities.

**Skills Demonstrated:** Account Lifecycle Management, Password Resets, IAM, Help Desk Administration, Kerberos Authentication.

---

## Case 06 — Secure SMB File Sharing & Access Control

### Objective

Configure an SMB departmental file share and restrict access using Active Directory security group membership.

### Scenario

An IT department requires a shared network folder that only authorized IT-Support group members can access.

Two domain users were used to test the security configuration.

| Domain User | IT-Support Membership | Expected Access |
|---|---|---|
| Sarah | Member | Allowed |
| James | Not a member | Denied |

### Steps Performed

#### Step 1 — Created the Shared Directory

Created the departmental directory on the Ubuntu Server.

```bash
sudo mkdir -p /srv/samba/IT-Department
```

Configured Linux filesystem ownership and permissions to support controlled access to the directory.

#### Step 2 — Configured the Samba Share

Added the departmental share configuration to:

```text
/etc/samba/smb.conf
```

Share configuration:

```ini
[IT-Department]
    path = /srv/samba/IT-Department
    browseable = yes
    read only = no
    valid users = @"NISHATLAB\IT-SUPPORT"
    force group = "NISHATLAB\IT-SUPPORT"
    create mask = 0660
    directory mask = 2770
```

The `valid users` setting restricted access to members of the specified Active Directory security group.

The configuration also defined file and directory creation permissions.

Validated the Samba configuration.

```bash
sudo testparm -s
```

**Screenshot 19 — Samba File Share Configuration**

![Samba File Share Configuration](Screenshots/19-samba-file-share-configuration.png)

#### Step 3 — Tested Authorized User Access

Attempted to connect to the departmental share using Sarah's domain account.

```bash
smbclient //127.0.0.1/IT-Department -U 'NISHATLAB\sarah'
```

After successfully connecting, uploaded a test file.

```text
put sarah-test.txt
```

Listed the share contents to verify the upload.

```text
ls
```

**Screenshot 20 — Authorized File Share Access**

![Authorized File Share Access](Screenshots/20-authorized-file-share-access.png)

The authorized account successfully accessed the share and uploaded the test file.

#### Step 4 — Tested Unauthorized User Access

Created a separate domain account named `james`.

James was not added to the IT-Support security group.

Attempted to access the departmental share.

```bash
smbclient //127.0.0.1/IT-Department -U 'NISHATLAB\james'
```

The connection was rejected with:

```text
NT_STATUS_ACCESS_DENIED
```

**Screenshot 21 — Unauthorized Share Access Denied**

![Unauthorized Share Access Denied](Screenshots/21-unauthorized-share-access-denied.png)

### Root Cause / Findings

The share's access restrictions were based on Active Directory security group membership.

Sarah was authorized through her membership in the IT-Support group, while James did not have the required membership.

The results demonstrated the distinction between authenticating as a domain user and being authorized to access a protected resource.

### Verification Results

| Security Test | Result |
|---|---|
| Authorized domain user connection | Successful |
| Authorized file upload | Successful |
| Uploaded file verification | Successful |
| Unauthorized user connection | Access denied |
| Group-based access restriction | Verified |

### Outcome

Successfully configured an SMB departmental share with Active Directory group-based access restrictions.

Verified that an authorized user could connect and upload a file, while an unauthorized domain user received an access-denied response.

This demonstrated practical implementation of the **principle of least privilege**.

**Skills Demonstrated:** SMB File Sharing, Active Directory Groups, Access Control, Linux Permissions, Least Privilege, Authorization Testing.

---

## Key Technical Skills Demonstrated

### Linux Server Administration

- Ubuntu Server installation and configuration
- Linux command-line administration
- Service management and troubleshooting
- Network configuration using Netplan
- System log analysis
- Virtual machine administration

### Active Directory & Identity Management

- Samba Active Directory Domain Controller
- Domain user provisioning
- Security group creation and administration
- Account disabling and re-enabling
- Password resets
- Kerberos authentication
- Identity and Access Management (IAM)

### Networking & Troubleshooting

- DNS A and SRV records
- LDAP and Kerberos service discovery
- DNS resolver configuration
- DNS forwarding investigation
- Port 53 conflict troubleshooting
- Samba DNS administration
- Network diagnostics

### Security & Access Control

- SMB/CIFS file sharing
- Active Directory group-based permissions
- Linux filesystem permissions
- Authentication and authorization
- Least-privilege access
- Authorized and unauthorized access testing

---

## Overall Project Outcome

Built and administered a Linux-based Active Directory environment using Ubuntu Server and Samba.

Across six case studies, the project demonstrated practical experience with:

1. Establishing a virtualized Linux server environment.
2. Troubleshooting Active Directory DNS and service discovery.
3. Verifying Kerberos domain authentication.
4. Creating and managing domain users and security groups.
5. Performing user account lifecycle administration.
6. Implementing and testing group-based SMB file sharing.

The strongest verified results were successful internal Active Directory DNS resolution, Kerberos authentication, domain user administration, and SMB access control.

External DNS forwarding through Samba remained inconsistent and is documented as an area for further troubleshooting.

### Professional Relevance

This project demonstrates foundational technical skills applicable to:

- IT Support Specialist
- Help Desk Technician
- Desktop Support Technician
- Junior System Administrator
- Identity & Access Management Support
- Entry-Level Infrastructure Support

---

## Future Improvements

Potential extensions to this lab include:

- Joining a compatible Windows client to the Active Directory domain.
- Managing domain accounts using graphical Active Directory tools.
- Exploring Group Policy administration in a Windows Server environment.
- Investigating and resolving the remaining external DNS forwarding issue.
- Configuring additional departmental SMB shares.
- Implementing file access auditing.
- Exploring automated user provisioning.
- Adding centralized logging and security monitoring.

---

**Project Type:** Personal Home Lab

**Focus Areas:** IT Support | System Administration | Identity & Access Management | Cybersecurity

*All activities were performed in a personal virtualized lab environment for educational and professional portfolio development.*
