# Linux Server & Active Directory Administration Lab

**Ubuntu Server | Samba Active Directory | DNS Troubleshooting | Kerberos | Identity & Access Management | SMB File Sharing**

## Project Overview

This project documents the deployment, administration, and troubleshooting of a Linux-based Active Directory environment using Ubuntu Server 24.04.4 LTS and Samba.

The lab simulates practical IT Support, Help Desk, and Junior System Administrator responsibilities, including domain user management, password resets, security group administration, authentication troubleshooting, DNS diagnostics, and secure file sharing.

The project consists of **six documented case studies**, each covering a technical objective or problem, the troubleshooting or implementation process, verification results, and screenshot evidence.

### Project Highlights

- Configured an Ubuntu Server virtual machine with Samba Active Directory Domain Controller services.
- Investigated DNS resolution issues and verified internal Active Directory DNS records.
- Successfully authenticated domain accounts using Kerberos.
- Created and managed Active Directory users and security groups.
- Performed user account disabling, re-enabling, and password resets.
- Configured an SMB departmental share with group-based access restrictions.
- Verified that an authorized user could access the share while an unauthorized user was denied.
- Documented troubleshooting findings, including an unresolved external DNS forwarding issue.

---

## Lab Environment

| Component | Configuration |
|---|---|
| Host Machine | Apple Silicon Mac (M4) |
| Virtualization | VMware Fusion |
| Guest Operating System | Ubuntu Server 24.04.4 LTS (ARM64) |
| Directory Service | Samba Active Directory Domain Controller |
| Samba Version | 4.19.5 |
| Server Hostname | dc01 |
| Active Directory Domain | nishatlab.local |
| Kerberos Realm | NISHATLAB.LOCAL |
| NetBIOS Domain | NISHATLAB |
| Network Mode | VMware NAT |
| Server IP Address | 192.168.204.130 |
| DNS | Samba Internal DNS |
| Authentication | Kerberos |
| File Sharing | SMB/CIFS |

### Tools & Technologies

- Ubuntu Server and Linux CLI
- VMware Fusion
- Samba Active Directory
- `samba-tool`
- `testparm`
- `systemctl` and `journalctl`
- `host` and `resolvectl`
- Netplan
- Kerberos (`kinit`, `klist`)
- `smbclient`
- Linux filesystem permissions

---

## Case Study Summary

| Case | Focus Area | Verified Result |
|---|---|---|
| 01 | Ubuntu Server & Active Directory Environment | Server environment established |
| 02 | Active Directory DNS Troubleshooting | Internal AD DNS verified; external forwarding remained inconsistent |
| 03 | Kerberos Authentication | Domain authentication and ticket verification successful |
| 04 | Active Directory Users & Groups | User and security group management successful |
| 05 | Account Lifecycle Management | Account changes and authentication verified |
| 06 | SMB File Sharing & Access Control | Authorized access successful; unauthorized access denied |

---

## Case 01 — Ubuntu Server & Active Directory Environment

### Objective

Establish a virtualized Ubuntu Server environment capable of supporting centralized domain authentication, identity management, and network file sharing.

### Implementation Steps

1. Installed Ubuntu Server 24.04.4 LTS (ARM64) using VMware Fusion.
2. Configured the virtual machine with NAT networking.
3. Established the server hostname `dc01`.
4. Installed and configured Samba Active Directory Domain Controller services.
5. Established the domain `nishatlab.local` and Kerberos realm `NISHATLAB.LOCAL`.
6. Verified the server environment and prepared it for domain administration tasks.

### Verification

The following commands can be used to inspect the environment and Samba configuration:

```bash
hostnamectl
sudo systemctl status samba-ad-dc
sudo testparm -s
```


**Ubuntu Server Version**

![Ubuntu Server Version](Screenshots/01-ubuntu-server-version.png)

### Outcome

Established a virtualized Ubuntu Server environment to support Samba Active Directory, internal DNS, Kerberos authentication, user management, and SMB file sharing.

**Skills Demonstrated:** Linux Server Administration, Virtualization, Samba Configuration, System Administration.

---

## Case 02 — Active Directory DNS Configuration & Troubleshooting

### Problem

During the Samba Active Directory configuration, DNS resolution issues affected domain service discovery.

Active Directory relies on DNS records to locate domain controllers and authentication services. The investigation focused on Samba DNS errors, resolver settings, port 53 conflicts, internal DNS records, and external forwarding behavior.

### Troubleshooting Steps

#### Step 1 — Investigated Samba DNS Errors

Reviewed Samba service logs to identify DNS-related errors.

```bash
sudo journalctl -u samba-ad-dc --no-pager
```

**Samba DNS Error Logs**

![Samba DNS Error Logs](Screenshots/02-samba-dns-error-logs.png)

#### Step 2 — Examined Initial DNS Resolver Settings

Inspected the server's resolver configuration.

```bash
resolvectl status
```

Reviewed which DNS servers were configured and how the operating system was resolving names.

**Initial DNS Server Settings**

![Initial DNS Server Settings](Screenshots/03-initial-dns-server-settings.png)

#### Step 3 — Investigated DNS Port 53 Conflicts

Checked active listeners on DNS port 53.

```bash
sudo ss -tulpn | grep ':53'
```

Investigated possible conflicts between Samba DNS and `systemd-resolved`.

**DNS Port 53 Conflict**

![DNS Port 53 Conflict](Screenshots/04-dns-port-53-conflict.png)

#### Step 4 — Verified Active Directory LDAP DNS Records

Queried the LDAP SRV record used for domain controller discovery.

```bash
host -t SRV _ldap._tcp.nishatlab.local ::1
```

**Active Directory LDAP DNS Record**

![Active Directory LDAP DNS Record](Screenshots/05-active-directory-ldap-dns-record.png)

#### Step 5 — Verified Kerberos DNS Records

Queried the Kerberos service discovery record.

```bash
host -t SRV _kerberos._udp.nishatlab.local ::1
```

**Kerberos DNS Record**

![Kerberos DNS Record](Screenshots/06-kerberos-dns-record.png)

#### Step 6 — Retested Internal DNS Resolution

Repeated internal DNS queries after investigating resolver and Samba DNS settings.

```bash
host -t A dc01.nishatlab.local ::1
host -t SRV _kerberos._udp.nishatlab.local ::1
```

**Kerberos DNS Resolution Verification**

![Kerberos DNS Resolution Verification](Screenshots/07-kerberos-dns-resolution-fixed.png)

#### Step 7 — Reviewed Netplan DNS Configuration

Inspected and adjusted the Ubuntu network configuration to address resolver behavior.

Validated the Netplan configuration.

```bash
sudo netplan generate
```

**Netplan DNS Configuration**

![Netplan DNS Configuration](Screenshots/08-netplan-dns-configuration.png)

#### Step 8 — Investigated External DNS Forwarding

Tested external DNS resolution through Samba and the upstream DNS server.

```bash
host -t A google.com 127.0.0.1
host -t A google.com 192.168.204.2
```

These tests helped distinguish local Samba forwarding issues from upstream DNS resolution.

**Samba DNS Forwarding Test**

![Samba DNS Forwarding Test](Screenshots/09-samba-dns-forwarding-success.png)

Some external DNS queries through Samba remained unsuccessful, even when direct upstream DNS resolution worked.

#### Step 9 — Verified Active Directory DNS Zone Records

Queried the Active Directory DNS zone using Samba's administration tool.

```bash
samba-tool dns query dc01.nishatlab.local nishatlab.local @ ALL -U Administrator
```

The query returned internal DNS zone information, including:

- Start of Authority (SOA) record
- Name Server (NS) record
- Domain Controller A record
- Active Directory DNS subdomains

**Active Directory DNS Zone Verification**

![Active Directory DNS Zone Verification](Screenshots/22-active-directory-dns-zone-verification.png)

The output confirmed that Samba could retrieve the internal Active Directory DNS zone records.

#### Step 10 — Verified Samba DNS Forwarder Configuration

Inspected Samba's configured DNS forwarder.

```bash
sudo samba-tool testparm --parameter-name="dns forwarder"
```

The output showed:

```text
192.168.204.2
```

**Samba DNS Forwarder Configuration**

![Samba DNS Forwarder Configuration](Screenshots/23-samba-dns-forwarder-configuration.png)

This confirmed the configured upstream DNS server. It did not independently establish that forwarding through Samba was working consistently.

### Findings

- Internal Active Directory DNS records were successfully verified.
- LDAP and Kerberos service discovery records were accessible.
- The Active Directory DNS zone query returned valid zone information.
- Samba's configured DNS forwarder was confirmed.
- External DNS forwarding through Samba remained inconsistent during testing.

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

Successfully investigated and verified internal Active Directory DNS functionality.

The troubleshooting process demonstrated practical experience with DNS records, Linux resolver configuration, service logs, port conflicts, Netplan, and Samba DNS administration.

External DNS forwarding was documented as an unresolved issue for future investigation rather than being presented as fully resolved.

**Skills Demonstrated:** DNS Troubleshooting, Linux Networking, Active Directory DNS, Netplan, Service Log Analysis, Samba Administration.

---

## Case 03 — Kerberos Authentication

### Objective

Verify that domain accounts could authenticate against the Samba Active Directory Domain Controller using Kerberos.

### Implementation & Verification Steps

#### Step 1 — Requested a Kerberos Ticket

Authenticated using the domain Administrator account.

```bash
kinit Administrator@NISHATLAB.LOCAL
```

This requested a Kerberos Ticket Granting Ticket (TGT).

#### Step 2 — Verified Successful Authentication

Inspected the Kerberos ticket cache.

```bash
klist
```

**Kerberos Authentication Success**

![Kerberos Authentication Success](Screenshots/10-kerberos-authentication-success.png)

#### Step 3 — Verified Kerberos Ticket Information

Reviewed the authenticated principal and ticket information.

```bash
klist
```

**Kerberos Authentication Verification**

![Kerberos Authentication Verification](Screenshots/11-kerberos-authentication-verification.png)

### Verification Results

| Test | Result |
|---|---|
| Domain Administrator authentication | Successful |
| Kerberos ticket request | Successful |
| Kerberos ticket cache verification | Successful |

### Outcome

Successfully authenticated a domain account through Kerberos and verified the resulting authentication ticket.

**Skills Demonstrated:** Kerberos Authentication, Domain Authentication, Identity Verification, Authentication Troubleshooting.

---

## Case 04 — Active Directory User & Security Group Management

### Objective

Simulate common IT Support responsibilities involving domain user provisioning and security group administration.

### Implementation Steps

#### Step 1 — Created a Domain User

Created an Active Directory user named `sarah`.

```bash
sudo samba-tool user create sarah
```

Verified the user account.

```bash
sudo samba-tool user show sarah
```

**Active Directory User Creation**

![Active Directory User Creation](Screenshots/12-active-directory-user-creation.png)

#### Step 2 — Created an Active Directory Security Group

Created a security group named `IT-Support`.

```bash
sudo samba-tool group add IT-Support
```

**Active Directory Security Group**

![Active Directory Security Group](Screenshots/13-active-directory-security-group.png)

#### Step 3 — Added the User to the Security Group

Assigned Sarah to the IT-Support group.

```bash
sudo samba-tool group addmembers IT-Support sarah
```

#### Step 4 — Verified Group Membership

Listed the members of the security group.

```bash
sudo samba-tool group listmembers IT-Support
```

**Security Group Membership**

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

Successfully created and managed an Active Directory user and security group.

The IT-Support group was subsequently used to control access to a departmental SMB share.

**Skills Demonstrated:** Active Directory Administration, User Provisioning, Security Group Management, Identity & Access Management.

---

## Case 05 — Active Directory User Account Lifecycle Management

### Objective

Simulate Help Desk account management tasks, including disabling accounts, re-enabling accounts, resetting passwords, and verifying authentication.

### Implementation Steps

#### Step 1 — Disabled a Domain User Account

Disabled Sarah's Active Directory account.

```bash
sudo samba-tool user disable sarah
```

Verified the account status.

**User Account Disabled**

![User Account Disabled](Screenshots/15-user-account-disabled.png)

#### Step 2 — Re-enabled the User Account

Restored Sarah's account to an enabled state.

```bash
sudo samba-tool user enable sarah
```

Verified the updated status.

**User Account Re-enabled**

![User Account Re-enabled](Screenshots/16-user-account-reenabled.png)

#### Step 3 — Reset the User Password

Performed an administrative password reset.

```bash
sudo samba-tool user setpassword sarah
```

**User Password Reset**

![User Password Reset](Screenshots/17-user-password-reset.png)

#### Step 4 — Verified Authentication After Password Reset

Authenticated using Sarah's updated credentials.

```bash
kinit sarah@NISHATLAB.LOCAL
klist
```

**User Kerberos Authentication**

![User Kerberos Authentication](Screenshots/18-user-kerberos-authentication.png)

### Verification Results

| Task | Result |
|---|---|
| Disable user account | Successful |
| Re-enable user account | Successful |
| Reset account password | Successful |
| Authenticate with updated credentials | Successful |
| Verify Kerberos ticket | Successful |

### Outcome

Successfully demonstrated user account lifecycle management, including account disabling, re-enabling, password resetting, and authentication verification.

These tasks reflect common account administration responsibilities performed by IT Support and Help Desk teams.

**Skills Demonstrated:** Account Lifecycle Management, Password Resets, IAM, Help Desk Administration, Kerberos Authentication.

---

## Case 06 — Secure SMB File Sharing & Access Control

### Objective

Configure a departmental SMB network share and restrict access using Active Directory security group membership.

### Scenario

An IT department requires a shared folder accessible only to authorized members of the IT-Support security group.

Two domain accounts were used to test the access restrictions.

| User | IT-Support Membership | Expected Access |
|---|---|---|
| Sarah | Member | Allowed |
| James | Not a member | Denied |

### Implementation Steps

#### Step 1 — Created the Shared Directory

Created a departmental directory.

```bash
sudo mkdir -p /srv/samba/IT-Department
```

Configured Linux filesystem ownership and permissions to support group-based access control.

#### Step 2 — Configured the Samba Share

Added the departmental share configuration to `/etc/samba/smb.conf`.

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

Validated the Samba configuration.

```bash
sudo testparm -s
```

**Samba File Share Configuration**

![Samba File Share Configuration](Screenshots/19-samba-file-share-configuration.png)

#### Step 3 — Tested Authorized User Access

Connected to the departmental share using Sarah's domain account.

```bash
smbclient //127.0.0.1/IT-Department -U 'NISHATLAB\sarah'
```

Uploaded a test file.

```text
put sarah-test.txt
```

Listed the contents of the share to verify the upload.

```text
ls
```

**Authorized File Share Access**

![Authorized File Share Access](Screenshots/20-authorized-file-share-access.png)

Sarah successfully accessed the departmental share and uploaded the test file.

#### Step 4 — Tested Unauthorized User Access

Created a second domain account named `james`.

James was not a member of the IT-Support security group.

Attempted to access the same SMB share.

```bash
smbclient //127.0.0.1/IT-Department -U 'NISHATLAB\james'
```

The connection was rejected.

```text
NT_STATUS_ACCESS_DENIED
```

**Unauthorized Share Access Denied**

![Unauthorized Share Access Denied](Screenshots/21-unauthorized-share-access-denied.png)

### Findings

The share's access restrictions were enforced through Active Directory security group membership.

- Sarah was authorized because she belonged to the IT-Support group.
- James was denied access because he did not belong to the authorized group.
- The results demonstrated the difference between domain authentication and resource authorization.

### Verification Results

| Test | Result |
|---|---|
| Authorized user connection | Successful |
| Authorized file upload | Successful |
| Uploaded file verification | Successful |
| Unauthorized user connection | Access denied |
| Group-based access restriction | Verified |

### Outcome

Successfully configured an SMB departmental share with group-based access restrictions.

Verified that an authorized domain user could access and upload files, while an unauthorized user received an access-denied response.

This demonstrated practical application of the **principle of least privilege**.

**Skills Demonstrated:** SMB File Sharing, Linux Permissions, Active Directory Security Groups, Access Control, Least Privilege, Authentication & Authorization.

---

## Key Technical Skills Demonstrated

### Linux Server Administration

- Ubuntu Server administration
- Linux command-line operations
- System service configuration
- Network configuration
- System log analysis
- Virtualization

### Active Directory & Identity Management

- Samba Active Directory Domain Controller
- User account provisioning
- Security group administration
- Account disabling and re-enabling
- Password resets
- Kerberos authentication
- Identity & Access Management

### Networking & Troubleshooting

- DNS A and SRV records
- LDAP and Kerberos service discovery
- DNS resolver troubleshooting
- Netplan configuration
- DNS forwarding investigation
- Port 53 diagnostics
- Samba DNS administration

### Security & Access Control

- SMB/CIFS file sharing
- Group-based permissions
- Linux filesystem permissions
- Least-privilege access
- Authentication and authorization testing
- Access restriction verification

---

## Overall Project Outcome

Built and administered a Linux-based Active Directory environment using Ubuntu Server and Samba.

Across six documented case studies, the project demonstrated hands-on experience with:

1. Linux server and Active Directory environment configuration.
2. Internal DNS troubleshooting and service discovery.
3. Kerberos authentication.
4. Domain user and security group administration.
5. Account lifecycle management.
6. SMB file sharing and group-based access control.

The strongest verified results were successful internal Active Directory DNS resolution, Kerberos authentication, user administration, and SMB access control.

External DNS forwarding through Samba remained inconsistent and is documented as an area for further investigation.


---

## Future Improvements

- Join a compatible Windows client to the Active Directory domain.
- Explore graphical Active Directory administration tools.
- Investigate and resolve the remaining external DNS forwarding issue.
- Configure additional departmental SMB shares.
- Implement file access auditing.
- Explore automated user provisioning.
- Add centralized logging and security monitoring.

---

**Project Type:** Personal Home Lab



*All activities were performed in a personal virtualized lab environment for educational and professional portfolio development.*
