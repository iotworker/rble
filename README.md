# R-BLE 使用说明

中文 | [English](./README.en.md)

> 由 Rust 实现的、面向 Linux / BlueZ 的 BLE 交互式命令行工具

**命令：** `rble`

## 概览

`rble` 是一个交互式 REPL 工具。启动后会进入提示符，直接输入命令即可对当前蓝牙适配器和设备执行操作。

## 快速上手

```bash
# 启动默认适配器
rble

# 指定适配器名或索引
rble --adapter hci1
rble --adapter 0
```

进入后可以直接执行常用操作：

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

## 启动参数

- `--adapter <name|index>` - 指定 BLE 适配器，支持名称或索引
- `--help` - 查看帮助
- `--version` - 查看版本

说明：

- 如果不指定 `--adapter`，默认使用第一个可用适配器
- 启动后会显示当前适配器名，连接后会额外显示当前设备 MAC

## 命令总览

### 设备发现
- `scan [OPTIONS]` - 扫描 BLE 设备
- `list [OPTIONS]` - 列出已缓存设备

### 连接与配对
- `connect <mac> [--skip-discovery]` - 连接设备
- `disconnect` - 断开当前设备
- `pair <mac>` - 与设备配对
- `unpair <mac>` - 取消配对
- `paired` - 列出已配对设备

### GATT 操作
- `info <mac>` - 查看设备信息
- `read <uuid> [--format ...]` - 读取特征值
- `write <uuid> <value> [--write-type ...]` - 写入特征值
- `notify <uuid> on [--format ...]` / `notify off` - 开启或关闭通知

### 帮助与退出
- `help [command]` - 查看帮助
- `quit` / `exit` - 退出

## 命令参考

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

参数：

- `--timeout <sec>` - 扫描时长，默认 `5`
- `--filter-name <name>` - 按设备名过滤
- `--filter-mac <mac>` - 按 MAC 过滤
- `--filter-service <uuid>` - 按服务 UUID 过滤
- `--sort rssi` - 按 RSSI 排序
- `--only-new` - 只显示本次新发现设备
- `--adv` - 显示更完整的广播信息
- `--continue` - 使用过滤条件时，不在第一个命中时停止扫描

说明：

- 默认情况下，如果加了过滤条件，`scan` 会在找到第一个匹配项后停止
- `--continue` 会覆盖这个行为

### `list`

```bash
R-BLE(hci0)> list
R-BLE(hci0)> list --sort rssi
R-BLE(hci0)> list --adv
```

参数：

- `--sort rssi` - 按 RSSI 排序
- `--adv` - 显示更完整的广播信息

说明：

- `list` 读取的是当前缓存中已知的设备，而不是重新发起扫描

### `connect`

```bash
R-BLE(hci0)> connect AA:BB:CC:DD:EE:FF
R-BLE(hci0)> connect AA:BB:CC:DD:EE:FF --skip-discovery
```

参数：

- `--skip-discovery` - 连接后跳过自动 GATT discovery

说明：

- 默认会在连接后自动发现 GATT
- 如果跳过 discovery，后续 `read` / `write` / `notify` 会在需要时再触发发现

### `pair` / `unpair`

```bash
R-BLE(hci0)> pair AA:BB:CC:DD:EE:FF
R-BLE(hci0)> unpair AA:BB:CC:DD:EE:FF
R-BLE(hci0)> paired
```

说明：

- `pair` 使用 BlueZ 的配对流程
- `paired` 会列出当前系统中已配对设备
- 某些需要 PIN / Passkey 输入的设备，当前交互模式无法手动输入，建议改用 `bluetoothctl`

### `info`

```bash
R-BLE(hci0)> info AA:BB:CC:DD:EE:FF
```

说明：

- 显示设备的基本信息
- 如果当前已经连接到同一个设备，会额外显示连接信息
- 当前连接可用时会展示 MTU、GATT service、characteristic、descriptor 统计和结构

### `read`

```bash
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> read 2a19
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> read 2a19 --format int-le
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> read 00002a19-0000-1000-8000-00805f9b34fb --format string
```

参数：

- `--format hex` - 以十六进制显示，默认值
- `--format string` - 以 UTF-8 字符串显示
- `--format int-le` - 以小端整数显示
- `--format int-be` - 以大端整数显示
- `--format raw` - 以原始字节形式显示

说明：

- `read` 读取前会确保当前连接的 GATT 已发现
- UUID 支持短 UUID 和完整 UUID

### `write`

```bash
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 0x64
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 text:hello
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 int:100
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 int-be:100
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> write 2a19 64 --write-type command
```

参数：

- `--write-type request` - 使用带响应写
- `--write-type command` - 使用不带响应写
- `--write-type auto` - 自动选择，默认值

值格式：

- `0xAABBCC` 或 `AABBCC` - 十六进制字节
- `AA BB CC` - 支持空格分隔的十六进制
- `text:hello` - UTF-8 文本
- `int:100` - 小端整数
- `int-be:100` - 大端整数

说明：

- 如果不写前缀，程序会优先按十六进制解析，失败后再按普通文本处理
- `write` 读取前会确保当前连接的 GATT 已发现

### `notify`

```bash
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify 2a37 on
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify 2a37 on --format string
R-BLE(hci0:AA:BB:CC:DD:EE:FF)> notify off
```

参数：

- `--format hex` - 以十六进制显示，默认值
- `--format string` - 以 UTF-8 字符串显示
- `--format raw` - 以原始字节形式显示

说明：

- `notify` 只支持当前已连接设备
- 同一时刻只保留一个活动订阅
- 切换到新的 characteristic 时，旧订阅会被自动取消

### `help`

```bash
R-BLE(hci0)> help
R-BLE(hci0)> help scan
R-BLE(hci0)> help write
```

说明：

- `help` 显示所有命令
- `help <command>` 显示单个命令的参数说明

## 适配器选择

`--adapter` 支持两种写法：

- 数字索引，例如 `0`
- 名称匹配，例如 `hci0`、`usb`

如果不指定，程序会使用系统中第一个可用的 Bluetooth 适配器。

## 权限与环境

### 系统要求

- Linux
- BlueZ
- `bluetoothd` 正在运行

### 常见权限配置

```bash
# 把当前用户加入 bluetooth 组
sudo usermod -aG bluetooth "$USER"

# 或者给二进制增加能力
sudo setcap 'cap_net_raw,cap_net_admin+eip' "$(which rble)"
```

某些系统还可能需要配置 polkit 规则，才能允许发现、连接和配对操作。

## 常见问题

### 找不到适配器

- 确认蓝牙硬件已启用
- 确认 `bluetoothd` 正在运行
- 用 `--adapter` 指定正确的索引或名称

### 无法扫描或连接

- 检查当前用户是否有蓝牙权限
- 尝试重新启动蓝牙服务：`sudo systemctl restart bluetooth`
- 某些环境需要额外的 polkit 或 capabilities 配置

### 配对失败

- 设备可能需要先进入配对模式
- 如果设备需要手动输入 PIN / Passkey，建议使用 `bluetoothctl`

### 读写失败

- 先确认已经连接到目标设备
- 如使用 `--skip-discovery`，先执行一次会触发 GATT 发现的操作
- 检查 UUID 是否正确，短 UUID 和完整 UUID 都支持
