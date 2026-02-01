# Remote Development Environment Setup Guide: MacOS Connecting to Windows Workstation

This guide details how to use VS Code Remote-SSH combined with the Tailscale intranet tunneling tool to remotely control a Windows workstation from a MacOS client for Python development.

## Phase 1: Building a Virtual LAN (Tailscale)

Tailscale is used to create a virtual local area network connecting devices located in different physical networks (e.g., home network and office intranet), solving issues related to missing public IPs and firewall blocking.

### 1. Windows Workstation Setup
1.  Visit the [Tailscale Download Page](https://tailscale.com/download/windows).
2.  Download and install the Windows client.
3.  Launch the application and log in using a Google, Microsoft, or GitHub account.
4.  Find the Tailscale icon in the system tray (bottom right), right-click it, and note down the displayed IP address (usually in the format `100.x.y.z`). This will be used as the **HostName** later.

### 2. MacOS Client Setup
1.  Download and install Tailscale via the Mac App Store or the official website.
2.  Launch and log in with the **exact same** account used on the Windows side.
3.  Ensure the status shows as **Connect**.

### 3. Connectivity Test
Run the following command in the MacOS terminal to verify if the network is connected:
```bash
ping 100.x.y.z
# Replace 100.x.y.z with the IP recorded from the Windows side
# If it returns "64 bytes from ...", the connection is successful
```

## Phase 2: Deploying SSH Service (Windows Side)

We will use the latest version of Win32-OpenSSH released on GitHub for installation, as this method is more stable and up-to-date than the optional feature built into Windows.

### 1. Download and Installation
1. Visit the GitHub release page: [PowerShell/Win32-OpenSSH Releases](https://github.com/PowerShell/Win32-OpenSSH/releases).
2. Download the latest `.msi` installer (e.g., `OpenSSH-Win64-v10.0.0.0.msi`).
3. Double-click the installer and follow the prompts to complete the installation.

### 2. Configure and Start the Service
Open PowerShell as Administrator (not Command Prompt) and execute the following commands in order:
```powershell
# 1. Start the SSHD service
Start-Service sshd

# 2. Set the service to start automatically on boot
Set-Service -Name sshd -StartupType 'Automatic'

# 3. Confirm service status (should show Running)
Get-Service sshd

# 4. Configure firewall to allow inbound traffic on port 22
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) { 
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22 
}
```

### 3. Get Login Username
Execute the following command in PowerShell to get the accurate username:
```powershell
whoami
# Example output: DESKTOP-PC\Administrator
# The username is the part after the backslash: Administrator
```

## Phase 3: VS Code Configuration (MacOS Side)

### 1. Install Extensions
1. Open VS Code.
2. Go to the Extensions Marketplace.
3. Search for and install **Remote - SSH**.

### 2. Configure SSH Connection Info
1. Press `Cmd + Shift + P` to open the Command Palette.
2. Type and select **Remote-SSH: Open Configuration File**.
3. Select the file at path `/Users/your_username/.ssh/config`.
4. Add the following configuration block:
```ssh
Host work-pc                # Enter a custom name
    HostName 100.x.y.z      # Enter the Tailscale IP of Windows
    User your_username      # Enter the username obtained via whoami
    Port 22
```

## Phase 4: Establishing Connection & Environment Setup

### 1. Initiate Connection
1. Click the blue remote icon `><` in the bottom left corner of VS Code.
2. Select **Connect to Host...**.
3. Choose the name defined in the Config (e.g., `work-pc`).

### 2. Enter Password
Use the PIN code or password of the Windows computer.

### 3. Remote Environment Mounting
1. Open the Extensions Marketplace, and under the **SSH: work-pc** section, install the Python extension.
2. Open any `.py` file.
3. Click the Python version number in the bottom right corner and select an existing environment on Windows.

## Phase 5: Troubleshooting

### VS Code Server Startup Crash
1. Run `cmd.exe` as Administrator on Windows.
2. Right-click the window title bar and select **Properties**.
3. Uncheck **Use legacy console**.
4. Delete the `C:\Users\username\.vscode-server` folder on Windows and retry.
