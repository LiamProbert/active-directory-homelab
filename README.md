# Active Directory Homelab

A Windows Server 2022 Active Directory environment running Fedora Linux, using KVM/QEMU virtualisation. The lab simulates a small office domain with a domain controller, a Windows 10 client, organisational units, domain users, and Group Policy enforcement.

## Lab Environment

| Component | Details |
|-----------|---------|
| Host Machine | Lenovo ThinkPad T480, Fedora Linux |
| Hypervisor | KVM / QEMU / virt-manager |
| Domain Controller (DC01) | Windows Server 2022 Standard Evaluation (Desktop Experience) |
| Client (Win10) | Windows 10 Pro |
| Domain | lab.local |
| Network | Default NAT (192.168.122.0/24) |

### VM Specifications

Each VM was configured with 4GB RAM, 2 CPU cores, and 40GB storage. Boot order was set in virt-manager with SATA CDROM first to allow installation from ISO.

## Domain Controller Setup (DC01)

### OS Installation

Windows Server 2022 Standard Evaluation (Desktop Experience) was installed using a custom install on a blank disk — no upgrade path existed, so a clean install was the only option.

### Static IP Configuration

A static IP was configured on DC01 to ensure domain-joined machines can always locate it, avoiding issues when a DHCP lease expires.

| Setting | Value |
|---------|-------|
| IP Address | 192.168.122.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.122.1 |
| Preferred DNS | 127.0.0.1 |

This was configured through Network Adapter Properties → Internet Protocol Version 4 (TCP/IPv4).

### Active Directory Domain Services

AD DS was installed through Server Manager using Add Roles and Features. After installation, the server was promoted to a domain controller by selecting "Add a new forest" and setting the root domain name to **lab.local**. A DSRM password was configured and DNS delegation was skipped.

Domain was verified as operational using `Get-ADDomain` in PowerShell.

### VirtIO Guest Tools

VirtIO Guest Tools were installed on DC01 to enable clipboard sharing between the Fedora host and the VM — useful for pasting PowerShell commands from a browser.

## Organisational Units and Users

Three OUs were created to represent departments in a typical office environment:

```powershell
New-ADOrganizationalUnit -Name "IT" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Workstations" -Path "DC=lab,DC=local"
```

Two domain users were created and placed into their respective OUs:

```powershell
# IT department user
New-ADUser -Name "Liam Probert" -SamAccountName "lprobert" `
  -UserPrincipalName "lprobert@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "LabPassword" -AsPlainText -Force) `
  -Enabled $true

# HR department user
New-ADUser -Name "Toby" -SamAccountName "Toby" `
  -UserPrincipalName "Toby@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "LabPassword" -AsPlainText -Force) `
  -Enabled $true
```

## Joining the Client to the Domain

On the Windows 10 Pro VM, the preferred DNS server was changed to **192.168.122.10** (DC01's static IP). The IP address was left on automatic since the client doesn't need a static address.

The machine was then joined to the domain via System Properties → Computer Name → Change → Domain: **lab.local**, authenticating with `LAB\Administrator`.

## Group Policy

A GPO called **"Block Control Panel – HR"** was created and linked to the HR OU.

The policy was configured through Group Policy Management → lab.local → HR OU → Right-click → Create a GPO → Edit → User Configuration → Policies → Administrative Templates → Control Panel.

### Testing

After logging in as **Toby** (HR OU) on the Win10 client, access to Control Panel was blocked — confirming the GPO was applied correctly. Logging in as **lprobert** (IT OU) showed unrestricted access, confirming the policy was scoped to HR only.

## What This Lab Demonstrates

- Installing and configuring Windows Server 2022 as a domain controller
- Promoting a server and creating a new Active Directory forest
- Static IP configuration for reliable domain services
- Creating and structuring Organisational Units
- Creating domain user accounts with PowerShell
- Joining a Windows client to a domain
- Creating, linking, and testing Group Policy Objects
- Working with KVM/QEMU virtualisation on Linux

## Built With

- Fedora Linux (host)
- KVM / QEMU / virt-manager
- Windows Server 2022 Standard Evaluation
- Windows 10 Pro
- PowerShell
