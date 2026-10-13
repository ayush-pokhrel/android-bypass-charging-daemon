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
