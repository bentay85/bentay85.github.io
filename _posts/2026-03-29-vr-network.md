---
title: "Setting Up a Dedicated Router for Quest 2 via Internet Connection Sharing (ICS)"
---

# Setting Up a Dedicated Router for Quest 2 via Internet Connection Sharing (ICS)  
My PC receives internet via Wi-Fi, and I want to share that connection to a dedicated VR router using an Ethernet cable. My router doesn't have a built-in "Access Point Mode," but I could still make this work perfectly using Windows Internet Connection Sharing (ICS).

### The Concept  
My PC gets internet from the main router at home via Wi-Fi. My PC then shares that internet access through its Ethernet port with the dedicated VR router. The dedicated router acts as a "dumb" switch and wireless transmitter for the Quest 2. I get both the video stream for wireless PC VR and internet on the Quest 2 through the same dedicated VR router. 

### Step 1: Configure the Dedicated VR Router  
Connect PC to one of the LAN ports of the dedicated VR router using an Ethernet cable. Some LAN ports might be 100mbps so check the Ethernet Connection Status to ensure that it is Gigabit speeds. Log into the router's admin page (usually 192.168.1.1 or 192.168.0.1). When I enabled ICS, Windows automatically changes the Ethernet adapter's IP address to 192.168.137.1 , so I needed to change the router's IP address to 192.168.137.2. 

Warning: Once you save this, you will lose connection to the dedicated VR router's configuration page. I then enable ICS to have Windows automatically assigning my PC's Ethernet port a static IP address of 192.168.137.1.  

### Step 2: Configure Windows Internet Connection Sharing (ICS)  
On the PC, press Win + R, type ncpa.cpl, and hit Enter. Find the Wi-Fi adapter (the one with the internet connection). Right-click it and select Properties. Go to the Sharing tab. Check the box: "Allow other network users to connect through this computer's Internet connection."

Note: When you do this, Windows automatically assigns your Ethernet port a static IP address of 192.168.137.1.

### Step 3: Configure the rest of the settings in the Dedicated VR Router  
On the PC, type 192.168.137.2 into the browser to get back into the configuration page of the Dedicated VR router. Disable the DHCP Server to allow the PC to handle IP addresses. 

Set a unique WiFi SSID (e.g., "Quest2"). Use the 5GHz band only. Set the channel width to 80MHz (or 160MHz if supported).

### Step 4: Connecting the Quest 2  
On the Quest 2, go to Wi-Fi settings and connect to the new SSID (e.g., "Quest2"). The Quest 2 should now show "Connected" and have internet access. Launch Air Link, Virtual Desktop, or SteamLink.

### Step 5: May be necessary if Quest 2 cannot connect after PC restarts  
Press Win + R, type regedit, and hit Enter. Navigate to: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\SharedAccess Right-click in the right pane, select New > DWORD (32-bit) Value. Name it EnableRebootPersistConnection and set its value to 1. 
