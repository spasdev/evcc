# Build Procedure for evcc on Raspberry Pi (Debian Trixie 64-bit)

This document describes how to cross-compile evcc from Windows for Raspberry Pi running Debian Trixie (64-bit ARM64).

## Prerequisites

### Windows Build Machine
- **Go**: Version 1.21+ (tested with Go 1.25.1)
- **Node.js**: Version 18+ (tested with Node.js 22.19.0)
- **npm**: Version 9+ (tested with npm 10.9.3)
- **Git**: For cloning the repository

### Target System
- Raspberry Pi 3/4/5 (64-bit)
- Debian Trixie (testing) ARM64
- Or any Linux ARM64 system

## Build Steps

### Step 1: Clone the Repository

```powershell
git clone https://github.com/evcc-io/evcc.git
cd evcc
```

### Step 2: Install Node.js Dependencies

```powershell
npm ci
```

Wait for completion. This installs all frontend dependencies.

**Expected output:**
```
added 823 packages, and audited 824 packages in 34s
found 0 vulnerabilities
```

### Step 3: Build the UI Assets

```powershell
npm run build
```

Wait for completion. This compiles the Vue.js frontend.

**Expected output:**
```
vite v7.x.x building client environment for production...
✓ xxx modules transformed.
...
✓ built in ~25s
```

### Step 4: Cross-Compile for Raspberry Pi ARM64

```powershell
$env:CGO_ENABLED="0"
$env:GOOS="linux"
$env:GOARCH="arm64"
go build -v -tags=release -trimpath -ldflags="-s -w" -o evcc_linux_arm64
```

Wait for compilation to complete. This may take several minutes.

**Key environment variables:**
- `CGO_ENABLED=0`: Disables CGO for static binary
- `GOOS=linux`: Target operating system
- `GOARCH=arm64`: Target architecture (64-bit ARM)

**Build flags:**
- `-tags=release`: Use release build tags
- `-trimpath`: Remove file system paths from binary
- `-ldflags="-s -w"`: Strip debug symbols for smaller binary

### Step 5: Verify the Build

```powershell
ls evcc_linux_arm64
```

**Expected output:**
```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        dd.mm.yyyy     hh:mm      ~104MB evcc_linux_arm64
```

## Deployment to Raspberry Pi

### Step 6: Copy Binary to Raspberry Pi

```powershell
scp evcc_linux_arm64 pi@<raspberry-pi-ip>:/home/pi/evcc
```

Or use any other file transfer method (USB, SMB, etc.)

### Step 7: Make Executable and Run on Raspberry Pi

SSH into the Raspberry Pi:

```bash
ssh pi@<raspberry-pi-ip>
```

Make the binary executable:

```bash
chmod +x /home/pi/evcc
```

Run evcc:

```bash
./evcc
```

## Optional: Create systemd Service

Create a service file on the Raspberry Pi:

```bash
sudo nano /etc/systemd/system/evcc.service
```

Add the following content:

```ini
[Unit]
Description=evcc - EV Charge Controller
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/home/pi/evcc --config /home/pi/evcc.yaml
WorkingDirectory=/home/pi
Restart=always
RestartSec=5
User=pi

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable evcc
sudo systemctl start evcc
```

Check status:

```bash
sudo systemctl status evcc
```

## Troubleshooting

### Binary won't execute
- Verify the architecture: `file evcc_linux_arm64` should show "ARM aarch64"
- Ensure executable permission: `chmod +x evcc_linux_arm64`

### Missing UI assets
- Make sure `npm run build` completed successfully before Go build
- The UI assets are embedded in the Go binary during compilation

### Build errors
- Run `go mod download` to ensure all Go dependencies are available
- Check Go version with `go version` (requires 1.21+)

## Version Information

- Build Date: December 2025
- Tested with:
  - Windows 11
  - Go 1.25.1
  - Node.js 22.19.0
  - npm 10.9.3
  - Target: Debian Trixie ARM64 (Raspberry Pi)
