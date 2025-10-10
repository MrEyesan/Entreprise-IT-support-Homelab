#  Complete Lab Setup Guide

> Step-by-step instructions to recreate this enterprise IT home lab from scratch.

---

## Prerequisites

### **Requirements:**
-  Microsoft Azure account (free tier available)
-  Credit card for verification (no charges if staying in free tier)
-  Computer with internet browser
-  Time investment: 20-30 hours over 4-6 weeks
-  Email address (for account signups)

**Estimated Costs:**
- Azure: $0-70/month depending on VM usage (deallocate when not in use)
- Microsoft 365: $0 (30-day E3 trial)
- Jira: $0 (free tier for up to 10 users)
- osTicket: $0 (open-source)
- **Total: $0-70/month** (minimize by stopping VMs when not actively using)

---

## Phase 1: Azure Infrastructure Setup

### **Step 1: Create Azure Account**

1. **Go to:** https://azure.microsoft.com/free/
2. **Click "Start free"**
3. **Sign in** with Microsoft account (or create new)
4. **Verify identity:**
   - Phone verification
   - Credit card (for identity only, not charged)
5. **Complete profile** information
6. **Agree** to terms and conditions

**Free Credits:**
- $200 credit for first 30 days
- 12 months of popular services free
- 55+ services always free

---

### **Step 2: Create Resource Group**

```bash
# Azure Portal method:
1. Search for "Resource groups"
2. Click "+ Create"
3. Subscription: (Your subscription)
4. Resource group name: HomeLabRG
5. Region: East US (or closest to you)
6. Click "Review + create" → "Create"
```

**Why Resource Groups?**
- Organizes all lab resources in one place
- Easy to manage permissions
- Simple cleanup (delete entire group when done)
- Cost tracking by resource group

---

### **Step 3: Create Virtual Network**

```bash
# In Azure Portal:
1. Search "Virtual networks"
2. Click "+ Create"
3. Resource group: HomeLabRG
4. Name: HomeLabVNet
5. Region: East US (same as resource group)
6. IP address space: 10.0.0.0/24
7. Subnet name: default
8. Subnet range: 10.0.0.0/24
9. Click "Review + create" → "Create"
```

**Network Planning:**
- 10.0.0.0/24 provides 254 usable IPs
- Sufficient for small lab environment
- Easy to remember IP scheme

---

### **Step 4: Deploy Windows Server VM (Domain Controller)**

**VM Configuration:**

```yaml
Basics:
  Resource group: HomeLabRG
  VM name: WIN-SERVER-DC
  Region: East US
  Availability: No infrastructure redundancy
  Image: Windows Server 2022 Datacenter - x64 Gen2
  Size: Standard_B2s (2 vCPU, 4GB RAM)
  
Administrator Account:
  Username: azureuser
  Password: YourStrongPassword123!
  
Inbound Ports:
  - RDP (3389)

Disks:
  OS disk type: Standard SSD
  Size: 128 GB

Networking:
  Virtual network: HomeLabVNet
  Subnet: default
  Public IP: (new) WIN-SERVER-DC-ip
  NIC network security group: Basic
  Public inbound ports: RDP (3389)
  
Management:
  Boot diagnostics: Disable (saves cost)
  
Tags:
  Environment: Lab
  Purpose: DomainController
```

**Important:** Write down the public IP address after deployment!

**Monthly Cost:** ~$30 (when running), ~$3 (when stopped/deallocated)

---

### **Step 5: Deploy Windows 10 Client VM**

```yaml
Basics:
  Resource group: HomeLabRG
  VM name: WIN10-CLIENT
  Region: East US
  Image: Windows 10 Enterprise, version 22H2 - x64 Gen2
  Size: Standard_B2s (2 vCPU, 4GB RAM)
  
Administrator Account:
  Username: azureuser
  Password: YourStrongPassword123!
  
Networking:
  Virtual network: HomeLabVNet (same as DC)
  Subnet: default
  Public IP: (new) WIN10-CLIENT-ip
  
All other settings: Same as Domain Controller
```

---

### **Step 6: Deploy Ubuntu Server VM**

```yaml
Basics:
  Resource group: HomeLabRG
  VM name: UBUNTU-TICKETING
  Region: East US
  Image: Ubuntu Server 22.04 LTS - x64 Gen2
  Size: Standard_B1s (1 vCPU, 1GB RAM)
  
Administrator Account:
  Authentication type: Password
  Username: ticketadmin
  Password: YourStrongPassword123!
  
Inbound Ports:
  - SSH (22)
  - HTTP (80)
  - HTTPS (443)
  
Networking:
  Virtual network: HomeLabVNet
  Subnet: default
  Public IP: (new) UBUNTU-TICKETING-ip
```

**Monthly Cost:** ~$8 (when running), ~$1 (when stopped)

---

### **Step 7: Configure Static IPs**

**For each VM:**

1. **Azure Portal** → Virtual machines → Select VM
2. **Networking** → Click network interface name
3. **IP configurations** → ipconfig1
4. **Assignment:** Change from Dynamic to **Static**
5. **Save**

**Recommended IP Assignment:**
- WIN-SERVER-DC: 10.0.0.4
- WIN10-CLIENT: 10.0.0.5
- UBUNTU-TICKETING: 10.0.0.6

---

## Phase 2: Active Directory Setup

### **Step 8: Connect to Domain Controller**

```bash
# Get public IP from Azure Portal
1. Virtual machines → WIN-SERVER-DC
2. Copy "Public IP address"
3. Open Remote Desktop Connection
4. Computer: [Public IP]
5. Username: azureuser
6. Password: YourStrongPassword123!
7. Click "Connect"
```

---

### **Step 9: Install Active Directory Domain Services**

**On WIN-SERVER-DC:**

```powershell
# Open PowerShell as Administrator

# Install AD DS Role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# This takes 5-10 minutes
```

---

### **Step 10: Promote to Domain Controller**

```powershell
# Promote server to domain controller
Install-ADDSForest `
  -DomainName "homelab.local" `
  -DomainNetbiosName "HOMELAB" `
  -ForestMode "WinThreshold" `
  -DomainMode "WinThreshold" `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) `
  -Force:$true

# Server will restart automatically
# Wait 5-10 minutes for restart
```

**After Restart:**
- Login as: `homelab\azureuser` (not just azureuser)
- Password: Same as before

---

### **Step 11: Configure DNS Settings**

**Update Virtual Network DNS:**

1. **Azure Portal** → Virtual networks → HomeLabVNet
2. **DNS servers** → Custom
3. **Add DNS server:** 10.0.0.4 (Domain Controller private IP)
4. **Save**
5. **Restart all VMs** to apply DNS changes

---

### **Step 12: Create Organizational Units**

**On Domain Controller:**

```powershell
# Open Active Directory Users and Computers
# Or use PowerShell:

New-ADOrganizationalUnit -Name "IT_Department" -Path "DC=homelab,DC=local"
New-ADOrganizationalUnit -Name "Sales_Department" -Path "DC=homelab,DC=local"
New-ADOrganizationalUnit -Name "HR_Department" -Path "DC=homelab,DC=local"
New-ADOrganizationalUnit -Name "Workstations" -Path "DC=homelab,DC=local"
New-ADOrganizationalUnit -Name "Servers" -Path "DC=homelab,DC=local"
```

---

### **Step 13: Create User Accounts**

```powershell
# Create users in respective OUs

# IT Department
New-ADUser -Name "John Smith" -GivenName "John" -Surname "Smith" `
  -SamAccountName "jsmith" -UserPrincipalName "jsmith@homelab.local" `
  -Path "OU=IT_Department,DC=homelab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "TempPass123" -AsPlainText -Force) `
  -Enabled $true -PasswordNeverExpires $true

New-ADUser -Name "Sarah Johnson" -GivenName "Sarah" -Surname "Johnson" `
  -SamAccountName "sjohnson" -UserPrincipalName "sjohnson@homelab.local" `
  -Path "OU=IT_Department,DC=homelab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "TempPass123" -AsPlainText -Force) `
  -Enabled $true -PasswordNeverExpires $true

# Sales Department
New-ADUser -Name "Mike Davis" -GivenName "Mike" -Surname "Davis" `
  -SamAccountName "mdavis" -UserPrincipalName "mdavis@homelab.local" `
  -Path "OU=Sales_Department,DC=homelab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "TempPass123" -AsPlainText -Force) `
  -Enabled $true -PasswordNeverExpires $true

New-ADUser -Name "Lisa Brown" -GivenName "Lisa" -Surname "Brown" `
  -SamAccountName "lbrown" -UserPrincipalName "lbrown@homelab.local" `
  -Path "OU=Sales_Department,DC=homelab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "TempPass123" -AsPlainText -Force) `
  -Enabled $true -PasswordNeverExpires $true

# HR Department
New-ADUser -Name "Tom Wilson" -GivenName "Tom" -Surname "Wilson" `
  -SamAccountName "twilson" -UserPrincipalName "twilson@homelab.local" `
  -Path "OU=HR_Department,DC=homelab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "TempPass123" -AsPlainText -Force) `
  -Enabled $true -PasswordNeverExpires $true
```

---

### **Step 14: Create Security Groups**

```powershell
# Create security groups

New-ADGroup -Name "IT_Admins" -GroupScope Global -GroupCategory Security `
  -Path "OU=IT_Department,DC=homelab,DC=local"

New-ADGroup -Name "Help_Desk" -GroupScope Global -GroupCategory Security `
  -Path "OU=IT_Department,DC=homelab,DC=local"

New-ADGroup -Name "Sales_Team" -GroupScope Global -GroupCategory Security `
  -Path "OU=Sales_Department,DC=homelab,DC=local"

New-ADGroup -Name "HR_Team" -GroupScope Global -GroupCategory Security `
  -Path "OU=HR_Department,DC=homelab,DC=local"

# Add users to groups
Add-ADGroupMember -Identity "IT_Admins" -Members jsmith
Add-ADGroupMember -Identity "Help_Desk" -Members sjohnson
Add-ADGroupMember -Identity "Sales_Team" -Members mdavis,lbrown
Add-ADGroupMember -Identity "HR_Team" -Members twilson
```

---

### **Step 15: Join Windows 10 Client to Domain**

**On WIN10-CLIENT VM:**

1. **Connect via RDP** using public IP
2. **Login** with local admin (azureuser)
3. **Change DNS** to Domain Controller:
   ```
   Network Settings → Ethernet → Properties → IPv4
   Preferred DNS: 10.0.0.4
   ```
4. **Join Domain:**
   ```
   Settings → System → About → Rename this PC (advanced)
   Computer Name → Change
   Domain: homelab.local
   Username: homelab\azureuser
   Password: [password]
   ```
5. **Restart** computer
6. **Login** with domain account: homelab\jsmith / TempPass123

---

## Phase 3: Microsoft 365 Setup

### **Step 16: Sign Up for Microsoft 365 Trial**

1. **Go to:** https://www.microsoft.com/en-us/microsoft-365/enterprise/office-365-e3
2. **Click "Free trial"**
3. **Enter email:** Use personal email for trial
4. **Create tenant:**
   - Company name: HomeLabIT (or your choice)
   - Organization size: 1-24
   - Country: Your location
5. **Tenant domain:** yourlabname.onmicrosoft.com
6. **Create admin account:**
   - Username: admin@yourlabname.onmicrosoft.com
   - Password: Strong password
   - **Write this down!**
7. **Verify identity** (phone/email)
8. **Complete setup**

**Trial includes:**
- 30 days free
- 25 user licenses
- All E3 features (Exchange, Teams, SharePoint, etc.)

---

### **Step 17: Install Microsoft Entra Connect Cloud Sync**

**On Domain Controller:**

1. **Open browser** → Go to https://entra.microsoft.com
2. **Sign in** with admin@yourlabname.onmicrosoft.com
3. **Navigate:** Identity → Hybrid management → Microsoft Entra Connect
4. **Click** "Cloud sync"
5. **Download** provisioning agent
6. **Run installer:** AADConnectProvisioningAgentSetup.exe
7. **Sign in** when prompted (M365 admin credentials)
8. **Configure Service Account:**
   - Select: Create gMSA (recommended)
   - OR use: homelab\administrator
9. **Connect Directory:**
   - Domain: homelab.local
   - Username: administrator
   - Password: [your password]
10. **Complete installation**

---

### **Step 18: Configure Cloud Sync**

**In Entra Admin Center:**

1. **Go to:** Cloud sync → Configurations
2. **Click** "New configuration"
3. **Select domain:** homelab.local
4. **Sync scope:** All users (or select specific OUs)
5. **Enable:** Password hash synchronization
6. **Save configuration**
7. **Enable** the configuration (toggle switch)
8. **Wait 2-5 minutes** for first sync

**Verify Sync:**
1. **Go to:** Identity → Users → All users
2. **Look for** synced users (John Smith, Sarah Johnson, etc.)
3. **Check:** "On-premises sync" column should say "Yes"

---

### **Step 19: Assign Microsoft 365 Licenses**

**For each synced user:**

1. **Entra admin center** → Users → Select user
2. **Licenses** → Assignments
3. **Select location:** Your country
4. **Assign:** Office 365 E3
5. **Save**

**Repeat for all users you want to license**

---

## Phase 4: osTicket Installation

### **Step 20: Connect to Ubuntu Server**

```bash
# Use SSH from your computer or Azure Cloud Shell

ssh ticketadmin@[UBUNTU_PUBLIC_IP]
# Enter password when prompted
```

---

### **Step 21: Install LAMP Stack**

```bash
# Update system
sudo apt update
sudo apt upgrade -y

# Install Apache, MySQL, PHP
sudo apt install apache2 mysql-server php php-mysql php-gd \
  php-imap php-xml php-mbstring php-curl unzip -y

# Start services
sudo systemctl start apache2 mysql
sudo systemctl enable apache2 mysql

# Verify Apache is running
sudo systemctl status apache2
```

---

### **Step 22: Configure MySQL Database**

```bash
# Secure MySQL installation
sudo mysql_secure_installation
# Answer: N, Y, Y, Y, Y

# Create database and user
sudo mysql

# In MySQL prompt:
CREATE DATABASE osticket;
CREATE USER 'osticket'@'localhost' IDENTIFIED BY 'ticketpass123';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticket'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

### **Step 23: Download and Install osTicket**

```bash
# Download osTicket
cd /tmp
wget https://github.com/osTicket/osTicket/releases/download/v1.18.1/osTicket-v1.18.1.zip

# Extract to web directory
sudo unzip osTicket-v1.18.1.zip
sudo cp -R upload/* /var/www/html/
sudo chown -R www-data:www-data /var/www/html/

# Configure osTicket
cd /var/www/html/include
sudo cp ost-sampleconfig.php ost-config.php
sudo chmod 0666 ost-config.php

# Remove Apache default page
sudo rm /var/www/html/index.html
```

---

### **Step 24: Complete Web Installation**

**From Windows 10 Client:**

1. **Open browser**
2. **Go to:** http://10.0.0.6/setup/
3. **Fill in form:**
   ```
   Helpdesk Name: HomeLabIT Support
   Default Email: support@homelab.local
   
   Admin User:
   - First Name: Admin
   - Last Name: User
   - Email: admin@homelab.local
   - Username: admin
   - Password: P@ssw0rd123!
   
   Database:
   - MySQL Hostname: localhost
   - MySQL Database: osticket
   - MySQL Username: osticket
   - MySQL Password: ticketpass123
   ```
4. **Click "Install Now"**
5. **Wait** for installation to complete

**Secure Installation:**

```bash
# Back on Ubuntu terminal:
sudo chmod 0644 /var/www/html/include/ost-config.php
sudo rm -rf /var/www/html/setup/
```

---

### **Step 25: Access osTicket**

**Staff Panel:** http://10.0.0.6/scp/  
**User Portal:** http://10.0.0.6/

---

## Phase 5: Jira Service Management

### **Step 26: Create Jira Account**

1. **Go to:** https://www.atlassian.com/try/cloud/signup
2. **Enter email** and click "Sign up"
3. **Verify email** (check inbox)
4. **Create site:**
   - Site name: homelabit (or your choice)
   - URL: homelabit.atlassian.net
5. **Select:** "Deliver great service"
6. **Choose:** "Jira Service Management"
7. **Create team name:** IT Support Team
8. **Complete setup**

---

### **Step 27: Configure Service Project**

**In Jira:**

1. **Project Settings** (gear icon bottom left)
2. **Details:**
   - Name: IT Support Desk
   - Key: ITS
3. **Request Types** → Add these:
   - Password Reset
   - Software Installation
   - Hardware Issue
   - Network Problem
   - Access Request
   - Email Issue
   - New User Setup
   - Mobile Device

---

## Phase 6: Creating Realistic Tickets

### **Step 28: Populate osTicket with Tickets**

See [TICKETS.md](TICKETS.md) for complete ticket scenarios.

**Quick guide:**
1. Go to http://10.0.0.6/
2. Submit ticket as user
3. Login to /scp/ as admin
4. Respond to ticket professionally
5. Resolve and document

**Create at least 10-15 tickets covering:**
- Account lockouts
- Software installations
- Hardware issues
- Network problems
- Email issues

---

### **Step 29: Populate Jira with Different Tickets**

**Focus on cloud/modern scenarios:**
- Azure VM issues
- Microsoft 365 problems
- Intune deployments
- Hybrid identity sync
- Conditional Access
- Mobile device management

**Create 10-15 tickets** with professional resolutions

---

## Phase 7: Documentation & Portfolio

### **Step 30: Take Screenshots**

**Capture these:**
- ☐ Azure resource group overview
- ☐ All 3 VMs running
- ☐ Active Directory structure
- ☐ osTicket dashboard
- ☐ osTicket resolved tickets
- ☐ Jira dashboard
- ☐ Jira resolved tickets
- ☐ Microsoft 365 users list
- ☐ Entra Connect sync status
- ☐ Security groups
- ☐ Group policies

---

### **Step 31: Create GitHub Repository**

```bash
# Create new repo on GitHub
# Clone locally

# Commit and push
git add .
git commit -m "Initial commit: Enterprise IT Home Lab"
git push origin main
```

---

## Cost Management Tips

### **Minimize Azure Costs:**

**Daily shutdown routine:**
```bash
# Stop all VMs when not in use
1. Azure Portal → Virtual machines
2. Select all VMs
3. Click "Stop"
4. Confirm

# Cost when stopped: ~$10/month (storage only)
# Cost when running: ~$68/month
```

**Set up automatic shutdown:**
1. VM → Operations → Auto-shutdown
2. Enabled: Yes
3. Time: 6:00 PM (or your preference)
4. Time zone: Your timezone
5. Notification: Email 15 minutes before

**Budget alerts:**
1. Cost Management + Billing
2. Budgets → Create budget
3. Amount: $50/month
4. Alert at: 80% ($40)

---

## Troubleshooting Common Issues

### **Can't RDP to VMs**
- Check NSG inbound rules (port 3389 allowed?)
- Verify VM is running
- Try connecting from different network
- Check if public IP changed after restart

### **Domain join fails**
- Verify DNS points to DC (10.0.0.4)
- Ping DC from client (ping 10.0.0.4)
- Ensure both VMs in same virtual network
- Check firewall rules on DC

### **Cloud Sync not working**
- Verify agent is running on DC
- Check configuration is "Enabled"
- Review provisioning logs for errors
- Ensure P1 license or M365 E3 assigned

### **osTicket errors**
- Check Apache error logs: `sudo tail -f /var/log/apache2/error.log`
- Verify MySQL service running
- Check file permissions
- Clear browser cache

---

## Next Steps After Setup

✅ **Populate with tickets** - Create realistic scenarios  
✅ **Document everything** - Screenshots and writeups  
✅ **Create KB articles** - Write troubleshooting guides  
✅ **Practice daily** - Spend 1-2 hours per day  
✅ **Update resume** - Add project to resume  
✅ **Share on LinkedIn** - Post about your lab  
✅ **Apply for jobs** - Start job hunting with portfolio  

---

## Time Investment Breakdown

| Phase | Time Required | Difficulty |
|-------|---------------|------------|
| Azure Setup | 2-3 hours | Easy |
| Active Directory | 3-4 hours | Medium |
| Microsoft 365 | 2-3 hours | Easy |
| osTicket | 2-3 hours | Medium |
| Jira Setup | 1-2 hours | Easy |
| Creating Tickets | 10-15 hours | Medium |
| Documentation | 5-8 hours | Easy |
| **Total** | **25-38 hours** | - |

Spread over 4-6 weeks = **4-6 hours per week**

---

## Support & Resources

**If you get stuck:**
-  Check official documentation
- Ask on Reddit r/homelab or r/ITCareerQuestions
- YouTube has many tutorials
- Feel free to reach out (contact info in README)

**Recommended reading:**
- Microsoft Learn (free courses)
- CompTIA A+ study materials
- Azure documentation
- ITIL Foundation concepts

---

**Good luck building your lab!** 

Remember: This is a learning project. Make mistakes, break things, rebuild. That's how you learn.

---

**Last Updated:** October 2025  
**Tested On:** Azure Free Tier  
**Success Rate:** 100% when following steps

