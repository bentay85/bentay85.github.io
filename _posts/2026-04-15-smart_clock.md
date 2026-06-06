---
title: "Getting a Smart Clock to Turn on My TV"
---
# Getting a Smart Clock to Turn on My TV  
If you’ve ever repurposed an old living room TV to act as a computer monitor, you know the struggle: it doesn't behave like a dedicated monitor. When you power on your PC, the TV stays off, and when you shut down, the TV stays on until you manually turn it off.

My wife recently got me a [desk clock and calendar](https://manuals.plus/tuya/smart-ir-remote-control-with-temperature-humidity-sensor-manual) which has a Smart IR device built in. By leveraging the **tinytuya** Python library and some built-in Windows automation, I now have a TV that syncs perfectly with my PC power state.

### The Tech Stack    

To bridge the gap between Windows and the IR blaster, I used:

*   **The Smart IR Device**: A Tuya-compatible clock and IR blaster.
    
*   **Python & TinyTuya**: A library used to interact with the device locally.
    
*   **Batch Scripts**: To trigger the Python logic.
    
*   **Group Policy Editor (gpedit.msc)**: To run scripts at startup and shutdown.

### Device Setup  

Setup the Clock and Smart IR device normally using the Android app. You should be able to control the TV from your smartphone. However, what we want to do is to automatically power on and shut down the TV with the PC.
 
## The Tuya IoT Platform Setup

You must first link your devices to a Tuya Developer account. This only needs to be done once to retrieve the device local keys.

1.  Create a Developer Account: Go to [iot.tuya.com](https://iot.tuya.com) and sign up.

2. Create a new cloud project and you need to take down the Access ID/Client ID, Access Secret/Client Secret and region. 

3. Add the Smart IR device to the project using the QR code on the app.

### Python and tinytuya  
1. Install [python](https://www.python.org/downloads/) on your PC. Run the following code to install the tinytuya library and run the wizard to retrieve information about your smart IR device.
```
pip install tinytuya
python -m tinytuya wizard  
```  
2. Enter the API Key (Access ID/Client ID) and API Secret (Access Secret/Client Secret) when prompted. Choose to scan for devices. Enter the region of your server when you created the cloud project.  

3. Choose yes when prompted to Download DP Mappings and yes to poll local devices. 

4. Look into devices.json and find the id, key and IP address and version for your Smart IR device.  

### Writing the Python Scripts  

First you need to learn the base64-encoded string for the power button. Run the script below and point your TV remote at the Smart IR device and press the power button. Take down the raw IR code which will be used in the next script to power on the TV through python.

```
import tinytuya
import json
d = tinytuya.Device('fill in smart IR device ID', 'fill in smart IR device IP address address', 'fill in smart IR device key')
d.set_version('fill in smart IR device version')
payload = d.generate_payload(tinytuya.CONTROL, {'201':'{"control":"study"}'})
d.send(payload)
print("Smart IR Device is now in Learning Mode. Point your Samsung remote at it and press POWER.")
data = d.receive()
print("Received Raw IR Code:", data)
```  

The following script (saved as main.py) triggers the Smart IR device to send a signal to power on and off the TV. It connects to the device using a unique ID, IP address, and local key. It sets the protocol version to 3.5 and sends a base64-encoded IR string that corresponds to the TV's power button.
```
from tinytuya.Contrib import IRRemoteControlDevice

d = IRRemoteControlDevice('fill in smart IR device ID', 'fill in smart IR device IP address', 'fill in smart IR device key', control_type=1)
d.set_version('fill in smart IR device version')

ir_code = "Learned IR code from the button of your actual remote"

result = d.send_button(ir_code)
```
Running the script using `python main.py` should turn off the TV. With your screen off, you can press up to go to the previous command and enter to turn your TV back on.

### Creating the Automation Scripts  

To make this hands-off, I created two batch files to handle the startup and shutdown triggers. These scripts use the uv Python package manager to execute the code.  

This script runs when the computer starts up to ensure the TV turns on.  
```
@echo off  
cd /d "C:\path\to\your\project"  
python main.py   
```
This script runs during the Windows shutdown process. It can be configured with a flag (like --shutdown-mode) if your script needs to handle the "off" command differently.  
```
@echo off  
cd /d "C:\path\to\your\project"  
python main.py --shutdown-mode
```

### Integrating with Windows  
The final step is telling Windows exactly when to run these files. Rather than using Task Scheduler, I used the **Group Policy Editor** for better reliability during the boot and power-off sequences.

1.  Press Win + R, type **gpedit.msc**, and hit Enter.
    
2.  Navigate to **Computer Configuration** > **Windows Settings** > **Scripts (Startup/Shutdown)**.
    
3.  Open **Startup** and add your startup.bat file.
    
4.  Open **Shutdown** and add your shutdown.bat file.  

### IP Address Reservation  
Finally, you would want to reserve the IP address for the Smart IR device in your router so that the script will always be able to find the device by IP Address and power on / off the TV.  
This can be done in your router configuration page by address reservation ( typically found at Advanced -> DHCP Server -> Address Reservation or similar). You will need the MAC address and current IP address from devices.json and reserve the currect IP address for the MAC address of the Smart IR device.  

### The Result  
Now, my workstation feels like a professional setup. When I press the power button on my PC, the smart clock beams the IR signal across the desk, and the TV is ready by the time I reach the login screen. It’s a simple automation that makes using an old TV as a monitor much more seamless.
