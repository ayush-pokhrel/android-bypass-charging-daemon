# bypassd

A tiny Bash daemon that automatically enables OnePlus/OPlus Bypass Charging whenever an Android device connects over ADB.

## Features

- Automatically detects ADB device connections
- Enables bypass charging
- Sets a custom charging limit
- No root required
- Uses the same Settings.System keys as the official OnePlus Battery app

## Requirements

- adb
- USB debugging enabled
- OnePlus/OPlus device supporting Bypass Charging

## Installation

mkdir -p ~/.local/bin
cp bypassd ~/.local/bin/
chmod +x ~/.local/bin/bypassd

mkdir -p ~/.config/systemd/user
cp bypassd.service ~/.config/systemd/user/

systemctl --user daemon-reload
systemctl --user enable --now bypassd.service

## Configuration

Edit the script:

LIMIT=80
STATE=1

Supported charging limits:

20
40
60
80


---
*"Setup / Development Notes"**.
---
```md
# bypassd Development Notes

## 1. Created the bypassd script

Created a Bash daemon named `bypassd` that automatically enables OnePlus/OPlus Bypass Charging when an Android device connects through ADB.

The script:

- Runs continuously in the background
- Detects new ADB device connections
- Reads current bypass charging settings
- Compares current values with desired values
- Only writes settings if changes are needed

Configuration:

```bash
LIMIT=80
STATE=1
```

Where:

- `LIMIT` controls the charging limit
- `STATE=1` enables bypass charging
- `STATE=0` disables bypass charging

---

## 2. Tested the script locally

Created the file:

```bash
nvim bypass
```

Added executable permission:

```bash
chmod +x bypass
```

Tested execution:

```bash
./bypass
```

Output:

```text
[bypassd] Device connected.
[bypassd] Done.
```

This confirmed:

- ADB connection detection works
- The daemon loop works
- Android settings commands execute successfully

---

## 3. Renamed the project

Renamed the script from:

```text
bypass
```

to:

```text
bypassd
```

Reason:

`bypassd` follows the Unix daemon naming convention.

Examples:

```text
sshd
systemd
bluetoothd
NetworkManager
```

The `d` represents a background daemon/service.

---

## 4. Installed the script locally

Created local executable directory:

```bash
mkdir -p ~/.local/bin
```

Copied the script:

```bash
cp bypassd ~/.local/bin/
```

Added executable permission:

```bash
chmod +x ~/.local/bin/bypassd
```

The script is now stored at:

```text
~/.local/bin/bypassd
```

---

## 5. Added ~/.local/bin to PATH

Checked current PATH:

```bash
echo $PATH
```

Problem:

`~/.local/bin` was missing.

Added it to zsh configuration:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

Reloaded zsh:

```bash
source ~/.zshrc
```

Verified:

```bash
echo $PATH
```

Expected output:

```text
/home/user/.local/bin:/usr/local/bin:/usr/bin:...
```

Now the script can run globally:

```bash
bypassd
```

instead of:

```bash
~/.local/bin/bypassd
```

---

## 6. Verified ADB connection

Checked connected devices:

```bash
adb devices
```

Expected:

```text
List of devices attached
XXXXXXXX    device
```

The daemon requires:

- USB debugging enabled
- ADB authorization accepted

---

## 7. Verified Android bypass settings

Checked current values:

```bash
adb shell settings get system bypass_charge_value
```

and:

```bash
adb shell settings get system bypass_charge_state
```

Expected:

```text
80
1
```

Meaning:

```text
80  -> Charging limit set to 80%
1   -> Bypass charging enabled
```

---

## 8. Future systemd user service

After confirming stability, create:

```bash
~/.config/systemd/user/bypassd.service
```

Enable:

```bash
systemctl --user daemon-reload

systemctl --user enable --now bypassd.service
```

This will automatically start bypassd when logging into Linux.

---

## Technical Discovery

The project uses Android Settings.System keys:

```text
bypass_charge_state
bypass_charge_value
```

These were identified by inspecting OnePlus Battery application behavior.

Instead of relying on:

- Quick Settings tiles
- UI automation
- Screen clicking

The daemon directly modifies the same system settings used by the Battery application.

Advantages:

- Faster execution
- Less fragile after OxygenOS updates
- No root required
- Fully automated through ADB
```

Also, one small GitHub tip: keep the `Battery.apk` and `battery_src` analysis in a separate folder like:

```text
research/
├── Battery.apk
└── battery_src/
```

and keep the main project clean:

```text
bypassd/
├── bypassd
├── bypassd.service
├── README.md
└── LICENSE
```

That separation makes it look much more like an actual reverse-engineering project instead of a random script dump.
