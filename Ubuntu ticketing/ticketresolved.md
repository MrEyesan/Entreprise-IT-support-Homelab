##  Tickets Resolved

### **Complete Ticket List (11 Tickets)**

#### **Account Management (2 Tickets)**

**#1: Account Lockout - URGENT**
- **Priority:** High
- **User:** Mike Davis
- **Resolution Time:** 15 minutes
- **Issue:** Account locked after 5 failed login attempts
- **Root Cause:** Password expired (90-day policy)
- **Actions Taken:**
  - Unlocked account via `Unlock-ADAccount -Identity mdavis`
  - Reset password to temporary: TempPass123!
  - Configured "must change at next logon"
  - User education on password policy
- **Prevention:** Implemented password expiration email warnings (14-day notice)

**#8: Shared Drive Access Request**
- **Priority:** Normal
- **User:** Tom Wilson
- **Resolution Time:** 20 minutes
- **Issue:** User needs read/write access to Finance shared drive
- **Verification:** Manager approval confirmed (Karen Mitchell)
- **Actions Taken:**
  - Added twilson to Finance_Access security group
  - Granted read/write NTFS permissions to \\\\SERVER\\Finance
  - Tested access from user's workstation
  - Updated access control audit log
- **Documentation:** Access grant logged in compliance system

---

#### **Hardware Support (2 Tickets)**

**#3: Printer Not Responding**
- **Priority:** Normal
- **User:** John Smith
- **Resolution Time:** 25 minutes
- **Issue:** HR-PRINTER-01 showing offline, cannot print
- **Root Cause:** Loose network cable + stuck print spooler jobs
- **Actions Taken:**
  - Physical inspection: Reseated network cable at wall jack
  - Printer regained IP (DHCP renewed)
  - Restarted print spooler service
  - Cleared 7 stuck print jobs
  - Updated printer driver to latest version
  - Reserved IP 10.0.0.50 in DHCP
- **Prevention:** Secured cable with clip, added printer to monitoring

**#9: Monitor Flickering**
- **Priority:** Normal
- **User:** Lisa Brown
- **Resolution Time:** 45 minutes
- **Issue:** User's monitor flickering intermittently
- **Troubleshooting:**
  - Tested monitor with different computer (monitor OK)
  - Inspected cables: found DisplayPort cable damage near connector
  - Replaced with certified DP 1.4 cable
  - Updated graphics drivers to latest version
- **Testing:** Monitored for 30 minutes post-fix, no recurrence
- **Parts Used:** DisplayPort cable (Asset #CABLE-089)

---

#### **Software Support (3 Tickets)**

**#2: Software Installation Request**
- **Priority:** Normal
- **User:** Lisa Brown
- **Resolution Time:** 30 minutes
- **Issue:** User needs Microsoft Project for project management
- **Verification:** Manager approval confirmed
- **Actions Taken:**
  - Verified license availability (3 unused Project Plan 3 licenses)
  - Assigned license via M365 admin center
  - Guided installation via portal.office.com
  - Tested Project Professional functionality
  - Provided quick start training (15 minutes)
- **Software:** Microsoft Project Professional (M365 version)

**#11: Microsoft Teams Not Working**
- **Priority:** Normal
- **User:** Tom Wilson
- **Resolution Time:** 40 minutes
- **Issue:** Teams showing "We ran into a problem" error
- **Root Cause:** Corrupted Teams cache files
- **Actions Taken:**
  - Closed Teams completely (killed background processes)
  - Deleted cache folders: %appdata%\\Microsoft\\Teams
  - Reinstalled Teams from teams.microsoft.com/downloads
  - Tested meeting join functionality
  - Verified audio/video devices working
- **Prevention:** Enabled automatic updates to prevent cache corruption

**#4: Slow Computer Performance**
- **Priority:** Normal
- **User:** Mike Davis
- **Resolution Time:** 35 minutes
- **Issue:** Computer taking 5+ minutes to boot, applications slow
- **Diagnostics:**
  - Disk space: 3GB free (97% full) 
  - Startup programs: 15 unnecessary programs 
  - Fragmentation: 62% fragmented 
- **Actions Taken:**
  - Ran Disk Cleanup: freed 45GB (temp files, Windows update cache)
  - Disabled 8 unnecessary startup programs
  - Performed disk defragmentation (95% complete)
  - Updated system drivers
  - Malware scan: system clean
- **Results:** Boot time reduced from 5 min to 45 sec (85% improvement)

---

#### **Network Issues (2 Tickets)**

**#6: VPN Connection Failure**
- **Priority:** High
- **User:** Mike Davis
- **Resolution Time:** 25 minutes
- **Issue:** Remote user cannot connect to corporate VPN
- **Error:** "VPN connection failed - certificate error"
- **Root Cause:** VPN server certificate renewed, client had cached old cert
- **Actions Taken:**
  - Removed existing VPN connection profile
  - Created new VPN connection with updated settings
  - Tested connection: successful
  - Verified access to internal resources
- **Documentation:** Created KB article for certificate renewal process

**#10: Internet Connectivity Issues**
- **Priority:** Normal
- **User:** Mike Davis
- **Resolution Time:** 30 minutes
- **Issue:** User has network connection but no internet access
- **Diagnostics:**
  - Network adapter: Connected, 1 Gbps
  - IP configuration: Valid IP (10.0.0.25)
  - Gateway: Pingable (10.0.0.1)
  - DNS: Incorrect (pointed to decommissioned server)
- **Root Cause:** DNS manually changed during previous troubleshooting
- **Actions Taken:**
  - Updated DNS to correct servers (10.0.0.4, 8.8.8.8)
  - Flushed DNS cache: `ipconfig /flushdns`
  - Renewed IP: `ipconfig /renew`
  - Tested connectivity: successful
- **Documentation:** Updated network troubleshooting runbook

---

#### **User Provisioning (2 Tickets)**

**#7: New Employee Onboarding**
- **Priority:** Normal
- **User:** Sarah Johnson (requester for Jennifer Cooper)
- **Resolution Time:** 2 hours 15 minutes
- **Scope:** Complete IT setup for new employee starting Monday
- **Actions Taken:**
  1. **Active Directory:**
     - Created user account: jcooper
     - Username: jcooper
     - Email: jcooper@homelab.local
     - Password: NewHire2025! (must change on first login)
     - Department: Sales
     - Security groups: Sales_Team, All_Employees
     
  2. **Email Account:**
     - Provisioned Exchange mailbox (50GB)
     - Configured Outlook profile settings
     - Setup guide sent to manager
     
  3. **Network Access:**
     - Sales shared drive: read/write access
     - Intranet access: enabled
     - VPN account: created
     
  4. **Software:**
     - Microsoft 365 E3 license assigned
     - Adobe Acrobat Reader
     - Company VPN client
     - Microsoft Teams
     
  5. **Hardware:**
     - Laptop: Dell Latitude 5420 (Asset #LAP-145)
     - Docking station
     - Monitor: 24" Dell (Asset #MON-089)
     - Keyboard, mouse, headset
     
  6. **Phone:**
     - Extension: 2105
     - Voicemail configured
     
- **Next Steps:** Equipment delivered Friday, IT orientation Monday 9 AM
- **Status:** Complete and ready for Monday start date

**#5: Email Problems - Stuck in Outbox**
- **Priority:** Normal
- **User:** Lisa Brown
- **Resolution Time:** 15 minutes
- **Issue:** Emails stuck in Outbox, not sending
- **Root Cause:** Outlook in "Working Offline" mode
- **Actions Taken:**
  - Clicked Send/Receive tab → disabled "Work Offline"
  - Clicked "Send/Receive All Folders"
  - All pending emails sent successfully (12 messages)
  - Verified email account settings
  - Sent test email: delivered
- **User Education:** Explained offline mode, connection status indicators

---
