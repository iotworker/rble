# R-BLE Usage Guide

English | [中文](./README.md)

> A Rust-based BLE interactive command-line tool for Linux / BlueZ

**Command:** `rble`

## Overview

`rble` is an interactive REPL tool. After launch, it opens a prompt where you can operate the current Bluetooth adapter and device directly.

## Quick Start

```bash
# Start with the default adapter
rble

# Specify an adapter by name or index
rble --adapter hci1
rble --adapter 0
```

Once inside, you can run common commands:

```text
R-BLE(hci0)> scan --timeout 5
R-BLE(hci0)> list
R-BLE(hci0)> connect AA:BB:CC:DD:EE:FF
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> read 2a19 --format int-le
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify 2a37 on
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify off
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> disconnect
R-BLE(hci0)> quit
```

## Startup Options

- `--adapter <name|index>` - Select the BLE adapter by name or index
- `--help` - Show help
- `--version` - Show version

Notes:

- If `--adapter` is not specified, the first available adapter is used
- The prompt shows the current adapter name, and when connected it also shows the active device MAC

## Command Overview

### Discovery
- `scan [OPTIONS]` - Scan for BLE devices
- `list [OPTIONS]` - List cached devices

### Connection and Pairing
- `connect <mac> [--skip-discovery]` - Connect to a device
- `disconnect` - Disconnect from the current device
- `pair <mac>` - Pair with a device
- `unpair <mac>` - Remove pairing
- `paired` - List paired devices

### GATT Operations
- `info <mac>` - Show device information
- `read <uuid> [--format ...]` - Read a characteristic value
- `write <uuid> <value> [--write-type ...]` - Write a characteristic value
- `notify <uuid> on [--format ...]` / `notify off` - Enable or disable notifications

### Help and Exit
- `help [command]` - Show help
- `quit` / `exit` - Exit

## Command Reference

### `scan`

```bash
R-BLE(hci0)> scan
R-BLE(hci0)> scan --timeout 10
R-BLE(hci0)> scan --filter-name Sensor
R-BLE(hci0)> scan --filter-mac AA:BB:CC:DD:EE:FF
R-BLE(hci0)> scan --filter-service 180d
R-BLE(hci0)> scan --sort rssi
R-BLE(hci0)> scan --only-new
R-BLE(hci0)> scan --adv
R-BLE(hci0)> scan --filter-name Sensor --continue
```

Options:

- `--timeout <sec>` - Scan duration, default `5`
- `--filter-name <name>` - Filter by device name
- `--filter-mac <mac>` - Filter by MAC address
- `--filter-service <uuid>` - Filter by service UUID
- `--sort rssi` - Sort by RSSI
- `--only-new` - Show only newly discovered devices in the current scan
- `--adv` - Show fuller advertisement details
- `--continue` - Keep scanning after the first match when filters are used

Notes:

- By default, when filters are used, `scan` stops after the first match
- `--continue` overrides that behavior

### `list`

```bash
R-BLE(hci0)> list
R-BLE(hci0)> list --sort rssi
R-BLE(hci0)> list --adv
```

Options:

- `--sort rssi` - Sort by RSSI
- `--adv` - Show fuller advertisement details

Notes:

- `list` reads cached devices instead of starting a new scan

### `connect`

```bash
R-BLE(hci0)> connect AA:BB:CC:DD:EE:FF
R-BLE(hci0)> connect AA:BB:CC:DD:EE:FF --skip-discovery
```

Options:

- `--skip-discovery` - Skip automatic GATT discovery after connecting

Notes:

- By default, GATT discovery runs after connection
- If discovery is skipped, `read` / `write` / `notify` will trigger it when needed

### `pair` / `unpair`

```bash
R-BLE(hci0)> pair AA:BB:CC:DD:EE:FF
R-BLE(hci0)> unpair AA:BB:CC:DD:EE:FF
R-BLE(hci0)> paired
```

Notes:

- `pair` uses the BlueZ pairing flow
- `paired` lists devices paired on the current system
- Devices that require manual PIN / passkey entry may need `bluetoothctl`

### `info`

```bash
R-BLE(hci0)> info AA:BB:CC:DD:EE:FF
```

Notes:

- Shows basic device information
- If the current connection matches the device, connection details are included
- When connected, it can show MTU plus GATT service, characteristic, and descriptor counts and structure

### `read`

```bash
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> read 2a19
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> read 2a19 --format int-le
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> read 00002a19-0000-1000-8000-00805f9b34fb --format string
```

Options:

- `--format hex` - Show as hex, default
- `--format string` - Show as UTF-8 string
- `--format int-le` - Show as little-endian integer
- `--format int-be` - Show as big-endian integer
- `--format raw` - Show raw bytes

Notes:

- `read` ensures GATT discovery before reading
- UUIDs can be short UUIDs or full UUIDs

### `write`

```bash
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 0x64
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 text:hello
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 int:100
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 int-be:100
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 64 --write-type command
```

Options:

- `--write-type request` - Write with response
- `--write-type command` - Write without response
- `--write-type auto` - Automatic selection, default

Value formats:

- `0xAABBCC` or `AABBCC` - Hex bytes
- `AA BB CC` - Hex bytes with spaces
- `text:hello` - UTF-8 text
- `int:100` - Little-endian integer
- `int-be:100` - Big-endian integer

Notes:

- If no prefix is used, the program tries hex first, then plain text
- `write` ensures GATT discovery before writing

### `notify`

```bash
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify 2a37 on
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify 2a37 on --format string
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify off
```

Options:

- `--format hex` - Show as hex, default
- `--format string` - Show as UTF-8 string
- `--format raw` - Show raw bytes

Notes:

- `notify` only works on the currently connected device
- Only one notification subscription is active at a time
- Switching to a different characteristic automatically unsubscribes the previous one

### `help`

```bash
R-BLE(hci0)> help
R-BLE(hci0)> help scan
R-BLE(hci0)> help write
```

Notes:

- `help` shows all commands
- `help <command>` shows command-specific details

## Adapter Selection

`--adapter` supports two forms:

- Numeric index, such as `0`
- Name match, such as `hci0` or `usb`

If omitted, the first available Bluetooth adapter is used.

## Environment

### Requirements

- Linux
- BlueZ
- Running `bluetoothd`

### Common Permissions Setup

```bash
# Add the current user to the bluetooth group
sudo usermod -aG bluetooth "$USER"

# Or grant capabilities to the binary
sudo setcap 'cap_net_raw,cap_net_admin+eip' "$(which rble)"
```

Some systems may also need polkit rules to allow discovery, connection, and pairing.

## Troubleshooting

### No Adapter Found

- Make sure Bluetooth hardware is enabled
- Make sure `bluetoothd` is running
- Use `--adapter` to select the correct index or name

### Scan or Connection Fails

- Check whether the current user has Bluetooth permissions
- Try restarting Bluetooth: `sudo systemctl restart bluetooth`
- Some environments need extra polkit or capability configuration

### Pairing Fails

- The device may need to be put into pairing mode first
- If the device requires a manual PIN / passkey, use `bluetoothctl`

### Read or Write Fails

- Make sure you are connected to the target device
- If you used `--skip-discovery`, run an operation that triggers GATT discovery first
- Confirm that the UUID is correct; short and full UUIDs are supported
