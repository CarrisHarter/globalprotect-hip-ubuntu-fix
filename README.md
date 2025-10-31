How to Fix GlobalProtect VPN "Limited Access" on Ubuntu 24.04This guide provides the steps to resolve the "Limited Access" error when connecting to a GlobalProtect VPN (like Monash University's) on Ubuntu 24.04.The ProblemAfter successfully authenticating to the GlobalProtect VPN, the client connects but immediately shows a "Limited Access" or "Policy Enforced" message. You are unable to access any internal network resources, and all network traffic may be blocked.This is caused by your device failing the Host Integrity Profile (HIP) check required by the VPN gateway.The DiagnosisThe HIP check fails because the most common policy requires an Anti-Virus with Real-Time Protection (RTP) enabled.Ubuntu does not have an AV by default.Simply installing clamav is not enough. The GlobalProtect agent will detect it, but incorrectly report Real-Time Protection: No unless configured correctly.This guide will walk you through fixing the Anti-Virus requirement.The SolutionThe fix is to install the open-source antivirus ClamAV and, crucially, enable and start its separate on-access (real-time) scanning service, clamonacc.Step 1: Install ClamAV ComponentsFirst, install the main ClamAV scanner and the clamav-daemon, which is required for the real-time service to function.# Update your package lists
sudo apt update

# Install the scanner and the daemon
sudo apt install clamav clamav-daemon
Step 2: Configure the ClamAV Daemon (clamd.conf)The main daemon (clamd) needs to be configured to listen for requests from the real-time (clamonacc) service.Open the configuration file with a text editor:sudo nano /etc/clamav/clamd.conf
Scroll to the bottom of the file and add the following lines. This enables the on-access scanning features:# --- Settings for On-Access (Real-Time) Scanning ---

# Enable On-Access Scanning
ScanOnAccess yes

# Block access to malicious files (required for RTP)
OnAccessPrevention yes

# Define which directories to watch (your /home is a good start)
OnAccessIncludePath /home

# Exclude the clamav user to prevent scan loops and permission errors
OnAccessExcludeUname clamav
Save the file and exit the editor (Press Ctrl+O, then Enter, then Ctrl+X).Step 3: Enable and Start the Real-Time Service (clamonacc)This is the most important step. You must enable and start the separate clamav-clamonacc service, which is disabled by default.First, restart the main clamav-daemon to make sure it loads your new configuration:sudo systemctl restart clamav-daemon
Now, enable and start the clamonacc service. The --now flag starts it immediately and also enables it to start automatically on boot.sudo systemctl enable clamav-clamonacc --now
VerificationYou can confirm that both services are running and that GlobalProtect now detects them correctly.1. Check the ServicesUse systemctl to check the status of both services.systemctl status clamav-daemon
systemctl status clamav-clamonacc
Both should show Active: active (running).2. Check the GlobalProtect ClientDisconnect and reconnect to the GlobalProtect VPN.Once connected, open the GlobalProtect client.Click the "hamburger" menu (three lines) and go to Settings.Select the Host Information Profile (HIP) tab.Expand the Anti-Malware category.You should now see ClamAV listed, and most importantly:Real-Time Protection: YesYour connection should now have full access.
