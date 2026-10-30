---
title: "Turning On The Lights And Fan From My Phone's Homescreen"
---
# Turning On The Lights And Fan From My Phone's Homescreen

In a previous post I used a smart clock with a built-in Smart IR device, the **tinytuya** library and some Windows automation to switch my TV on and off with the PC: [link](https://bentay85.github.io/2026-04-15/smart-clock.html)

That setup works, but it runs from the PC, and the PC only handles what it was scripted to do. The next step was to control my lights (through the IR blaster) plus my Xiaomi smart fan directly from an icon on my phone's homescreen - no PC involved. Since both devices are on the LAN, the phone can talk to them just as well as the desktop does.

### The Tech Stack

*   **Termux and Termux:Widget**: a terminal and homescreen shortcut app on Android.
*   **tinytuya**: controls the Smart IR blaster to toggle the lamp.
*   **python-miio**: controls the Xiaomi smart fan over the LAN.
*   **WinSCP**: to copy the scripts from my PC to the phone over SSH.

### Getting Termux Onto The Phone

Termux is not on the Play Store in a working state, so I installed it the open-source way:

1.  Install the [F-Droid](https://f-droid.org/en/) app store from its website.
2.  From within F-Droid, install **Termux** and **Termux:Widget**.

### Copying Files Over: SSH On The Phone

To move files from Windows to the phone I ran an SSH server on the phone and connected with WinSCP. In Termux:

```
pkg install openssh
passwd
sshd
```

Set the password *before* starting the server, otherwise you cannot log in. Then find the phone's IP address - mine showed up on my router's homepage under connected devices - and check the username inside Termux with:

```
whoami
```

In WinSCP, choose SFTP, enter that username, the phone's IP and port **8022** (Termux's SSH runs on a high port), enter the password, and copy the project folder over to the phone. One gotcha: files copied from Windows arrive with CRLF line endings, which breaks `#!` shebang lines on Android. My install script fixes this with `sed -i 's/\r$//'` on every script it installs.

### Getting The Device Credentials

Two sets of credentials are needed, one per device:

*   **Smart IR blaster**: the device ID, local key and IP address for tinytuya. The whole Tuya IoT Platform + `python -m tinytuya wizard` process to retrieve these is covered in my smart clock post: [link](https://bentay85.github.io/2026-04-15/smart-clock.html). I reused the same values from `devices.json` on the PC.
*   **Xiaomi fan**: python-miio authenticates with a per-device *token*. I extracted mine with [Xiaomi-cloud-tokens-extractor](https://github.com/PiotrMachowski/Xiaomi-cloud-tokens-extractor) by logging in with my Xiaomi account.

Both secrets go in a `config.toml` next to the script, never in the code:

```toml
[fan]
ip = "192.168.0.101"
token = "fill in fan token"
model = "dmaker.fan.1c"

[ir]
ip = "192.168.0.100"
device_id = "fill in smart IR device ID"
local_key = "fill in smart IR local key"
protocol_version = "3.5"

[[ir.commands]]
name = "light power on/off"
code = "learned IR code from the lamp remote's power button"
```

### IP Address Reservation

Same lesson as the smart clock post: the phones' and PC's scripts find the devices by IP, so the IPs must never change. On my router I used **DHCP Address Reservation** to pin a static address for the Smart IR blaster and the fan (Advanced -> DHCP Server -> Address Reservation or similar). My fan ended up at `192.168.0.101` and the IR box at `192.168.0.100`.

### The Script: One Press, Both Devices

The heart of the project is a single Python script, `home_group.py`, with three actions:

```
~/automation/home_group.py on
~/automation/home_group.py off
~/automation/home_group.py status
```

The two devices behave differently, so the script handles them differently:

*   **The fan** takes explicit on/off commands over MiOT via python-miio. The script *reads* the fan's reported power first, only writes when it differs from what was asked, then re-reads to confirm. A repeated press is therefore harmless.
*   **The lamp** is driven by the IR blaster playing back the learned code from the lamp remote's single power button. Infrared is a blind toggle - nothing is acknowledged and nothing is remembered between presses, so each press sends the code exactly once. If the lamp ends up in the wrong state (it was already on when you pressed *On*), just press the same button again and it re-toggles, while the fan write is skipped because it already reports the state you asked for.

```python
import tinytuya
from miio import Fan1C

fan = Fan1C(fan_cfg["ip"], fan_cfg["token"], model="dmaker.fan.1c")
fan.on()  # or fan.off()

payload = json.dumps(
    {"control": "send_ir", "type": 0, "head": "", "key1": "1" + code},
    sort_keys=True,
)
device = tinytuya.OutletDevice(ir_cfg["device_id"], ir_cfg["ip"], ir_cfg["local_key"])
device.set_version("3.5")
device.set_value("201", payload, nowait=True)
```

The write to the IR box uses `nowait=True` because this box never answers a write, with a short sleep afterwards so it finishes emitting the signal before the socket closes.

An `install.sh` puts everything in place in one go: it copies `home_group.py` and `config.toml` to `~/automation/`, copies the shortcuts to `~/.shortcuts/` (where current Termux:Widget looks; older builds used `~/bin/`), strips any CRLF line endings, marks everything executable, and installs the Python dependencies:

```
pkg install python
pip install 'python-miio>=0.5.12,<0.6' 'tinytuya>=1.20.0,<2'
```

### The Homescreen Buttons

Termux:Widget runs any executable placed in `~/.shortcuts/`, so each button is a tiny bash wrapper around the script:

```bash
#!/data/data/com.termux/files/usr/bin/bash
out=$(/data/data/com.termux/files/usr/bin/python "$HOME/automation/home_group.py" on 2>&1); rc=$?
printf '%s\n' "$out"
if command -v termux-toast >/dev/null 2>&1; then
    if [ $rc -eq 0 ]; then termux-toast -s -b '#1d7a33' -fg white "$out"
    else termux-toast -s -b '#8f1d1d' -fg white "$out"; fi
fi
exit $rc
```

The `termux-toast` lines (from the Termux:API app plus `pkg install termux-api`) pop up a green or red notification showing what happened - handy for debugging, but the buttons still work without them, they just stay quiet.

To get the buttons onto the homescreen:

1.  Long-press the home screen > **Widgets** > **Termux:Widget**.
2.  Drag one out and pick **Lights + Fan On** from the shortcut list.
3.  Repeat for **Lights + Fan Off**. (Refresh the widget if new shortcuts are missing from the picker.)

### The Result

One press on the homescreen and the fan spins up while the IR blaster toggles the lamp - without the PC being on. Two small annoyances worth remembering: if the fan is ever re-paired its token rotates, and if the IR box is removed and re-added in the Tuya app its local key rotates, so both need re-copying into `config.toml`. And Android likes to kill background jobs, so exclude Termux from battery optimisation if presses fail with the screen off.
