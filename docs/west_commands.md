# Zephyr + West Developer Command Reference

This file collects the most useful `west` commands for working in a custom Zephyr workspace like this:

```
/west-workspace/
├── apps/
├── build/
├── documents/
├── manifest-repo/
├── modules/
├── venvs/
└── zephyr/
```

## 1. Workspace Setup

### Initialize a Workspace

```bash
west init -m https://github.com/zephyrproject-rtos/zephyr --mr main
west update
```

### Set your python instance

#### For Powershell
```bash
west config build.west-python "$PWD\.venv\Scripts\python.exe"

# Verify with
west config build.west-python
```

### Set a Different Manifest

```bash
west config manifest.path manifest-zephyrproject
west config manifest.file west.yml
west update
```

### Show Current Configuration

```bash
west config manifest.path
west config manifest.file
```

### Fix Stuck `west update` (force sync)

```bash
west update --reset-fetch
```

---

## 2. Building Applications

### Standard Build (new app or board)

If switching apps or boards, add `--pristine`:

```bash
west build -b <board> apps/<app_name> --pristine
```

Example:

```bash
west build -b max32655fthr apps/blinky_max32655FTHR --pristine
```

### Use a Dedicated Build Directory (preferred for multiple apps)

```bash
west build -b <board> -s apps/<app_name> -d build_<board_or_app>
```

Example:

```bash
west build -b esp32 -s apps/blinky_esp32 -d build_esp32
```

### Multi-Core Build

Linux/macOS:
```bash
west build -b <board> apps/<app_name> -j$(nproc) --pristine
```

PowerShell:
```powershell
west build -b <board> apps/<app_name> -j$(Get-ComputerInfo).CsNumberOfLogicalProcessors --pristine
```

---

### Rebuild Without Wiping

```bash
west build -t clean
west build
```

### Full Clean (manually remove build/)

Linux/macOS:
```bash
rm -rf build/
```

PowerShell:
```powershell
Remove-Item -Recurse -Force build
```

---

## 3. Flashing and Debugging

### Flash Target

```bash
west flash
```

### Flash with a Specific Runner (optional)

```bash
west flash --runner jlink
```

### Start Debugger

```bash
west debug
```

### Attach Debugger Without Reset

```bash
west debug --no-reset
```

---

## 4. Project Introspection

### List All Supported Boards

```bash
west boards
```

Limit by architecture:

```bash
west boards | grep esp32
```

### List All Known Runners

```bash
west flash --help
```

### Check Active Modules

```bash
west list
```

### Find Path of a Module

```bash
west list hal_espressif
```

### Custom Output Format

```bash
west list -f '{name}: {path}'
```

---

## 5. Advanced Build Options

### Force Reconfiguration

```bash
west build -p always
```

### Rebuild with Specific Toolchain

```bash
west build -b <board> -- -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake
```

---

## 6. Twister Unit Testing (Optional)

Run all tests:

```bash
west twister -T tests/
```

Filter by platform:

```bash
west twister -p native_posix
```

---

## 7. Manifest Management

### View Current Manifest (freeze version)

```bash
west manifest --freeze > west.yml
```

### List Projects in Manifest

```bash
west manifest --resolve
```

---

## 8. Virtual Environment Activation

### PowerShell (Windows)

```powershell
& venvs\zephyrproject\Scripts\Activate.ps1
```

Unblock if needed:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### MSYS2 / Git Bash (Windows)

```bash
source venvs/zephyrproject/Scripts/activate
```

Or:

```bash
source venvs/zephyrproject/bin/activate
```

---

## 9. Useful Environment Checks

### Check Logical Core Count

Linux/macOS:
```bash
nproc
```

PowerShell:
```powershell
(Get-ComputerInfo).CsNumberOfLogicalProcessors
```

### Show Top Directory

```bash
west topdir
```

---

## 10. Help and Docs

```bash
west --help
west build --help
west flash --help
```

Official docs: https://docs.zephyrproject.org/latest/develop/west/index.html
