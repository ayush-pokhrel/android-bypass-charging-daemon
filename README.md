# bypassd

A small Bash daemon that automatically enables **OnePlus / OPlus Bypass Charging** whenever your Android device connects over ADB.

Plug in your phone, and `bypassd` sets your charge limit and turns on bypass charging. No taps, no root, no UI automation.

## Features

- Detects ADB device connections automatically
- Enables bypass charging and applies a configurable charge limit (20 / 40 / 60 / 80 %)
- Writes only when a value differs, then reads it back to verify
- No root required
- Runs as a `systemd` user service and recovers from unplugs and reboots
- Configured through a plain config file, so there is no need to edit the script
- Uses the same `Settings.System` keys as the official OnePlus Battery app

## Requirements

- Linux with `systemd` (for the service; the script itself runs on any system with Bash)
- [`adb`](https://developer.android.com/tools/adb) available in `PATH`
- **USB debugging** enabled on the phone, with this computer's ADB key authorized
- A OnePlus / OPlus device that supports Bypass Charging

## Compatibility

| Device | OS version | Status |
| ------ | ---------- | ------ |
| OnePlus Pad 3 | OxygenOS 16.0.10 | Tested |

The `bypass_charge_state` and `bypass_charge_value` keys are undocumented and may change between OS releases. If it works (or doesn't) on your device, please open an issue or PR to update this table.

## Installation

```sh
git clone https://github.com/ayush-pokhrel/android-bypass-charging-daemon.git
cd android-bypass-charging-daemon

install -Dm755 bypassd         ~/.local/bin/bypassd
install -Dm644 bypassd.service ~/.config/systemd/user/bypassd.service

systemctl --user daemon-reload
systemctl --user enable --now bypassd.service
```

Confirm it is running:

```sh
systemctl --user status bypassd.service
journalctl --user -u bypassd.service -f
```

## Configuration

Settings are supplied through environment variables. To change them, create a config file:

```sh
install -Dm644 bypassd.conf.example ~/.config/bypassd/bypassd.conf
$EDITOR ~/.config/bypassd/bypassd.conf
systemctl --user restart bypassd.service
```

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `BYPASSD_LIMIT` | `80` | Charge limit in percent. One of `20`, `40`, `60`, `80`. |
| `BYPASSD_STATE` | `1` | `1` enables bypass charging, `0` disables it. |
| `BYPASSD_SERIAL` | _unset_ | ADB serial to target. Required only if several devices are attached (see `adb devices`). |
| `BYPASSD_CONNECT_DELAY` | `3` | Seconds to wait after a device connects before writing settings. |
| `BYPASSD_POLL_INTERVAL` | `2` | Seconds between connection checks. |

Invalid values cause the daemon to exit with a clear error (exit code 78), and systemd will not restart it until you fix the config.

## Usage

Once the service is enabled, there is nothing to do: connect your phone and check the log.

```text
[bypassd] started (limit=80%, state=1)
[bypassd] device connected
[bypassd] setting bypass_charge_value: 100 -> 80
[bypassd] setting bypass_charge_state: 0 -> 1
[bypassd] done
```

You can also run it in the foreground for testing:

```sh
BYPASSD_LIMIT=60 ./bypassd
```

### Verify on the device

```sh
adb shell settings get system bypass_charge_value   # e.g. 80
adb shell settings get system bypass_charge_state   # 1 = enabled
```

## How it works

OnePlus's Battery app stores its bypass-charging configuration in two `Settings.System` keys:

| Key | Meaning |
| --- | ------- |
| `bypass_charge_value` | Charge limit in percent |
| `bypass_charge_state` | `1` = bypass enabled, `0` = disabled |

`bypassd` polls `adb get-state`. When the device transitions to `device` (a fresh connection), it waits briefly for the device to settle, then reads each key and writes only the ones that differ, using `adb shell settings put system ...`. Because it talks to the settings provider directly rather than simulating taps, it is fast and unaffected by UI changes.

## Troubleshooting

**Nothing happens when I plug in the phone.**
Run `adb devices`. The device must be listed as `device`, not `unauthorized` or `offline`. Accept the USB debugging prompt on the phone.

**`adb not found in PATH` in the logs.**
systemd user services use a minimal `PATH`. If `adb` lives somewhere unusual, add `Environment=PATH=/your/adb/dir:/usr/local/bin:/usr/bin:/bin` to the `[Service]` section of the unit file, then run `systemctl --user daemon-reload`.

**`multiple devices attached` warning.**
Set `BYPASSD_SERIAL` in your config to the serial shown by `adb devices`.

**Settings are written but bypass charging doesn't engage.**
Your OS version may use different keys, or the feature may need to be enabled once in the Battery app. Please open an issue with your device model, OS version, and the output of `adb shell settings list system | grep -i bypass`.

## Uninstall

```sh
systemctl --user disable --now bypassd.service
rm ~/.config/systemd/user/bypassd.service ~/.local/bin/bypassd
rm -rf ~/.config/bypassd
systemctl --user daemon-reload
```

## Limitations

- Settings are applied when a device **connects**. If the phone stays connected and you change the setting on the device, `bypassd` will not override it until the next reconnect.
- Depends on undocumented OnePlus/OPlus settings keys that the vendor may change.
- Requires an ADB connection (USB or wireless) to the machine running the daemon.

## Disclaimer

This is an unofficial community project and is not affiliated with or endorsed by OnePlus or OPlus. It modifies system settings on your device; use it at your own risk.

## Contributing

Issues and pull requests are welcome, especially compatibility reports for additional devices and OS versions. Please run `shellcheck bypassd` before submitting changes.

## License

[MIT](LICENSE)
