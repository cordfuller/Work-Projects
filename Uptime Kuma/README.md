# Uptime Kuma work set-up
## Why I Built This
We lost internet for a few minutes one day. When I asked a coworker if he had
noticed, he had no idea it had even happened. I wanted a way
to get notified the moment something went down, so we would know about it before
it became a bigger issue instead of finding out after the fact.

That turned into this project: a monitoring stack that watches our network
from the inside out and alerts me when any part of the chain breaks.

A project that I setup to monitor our servers and devices if interruptions happen it can notify me. the way the stack will be built is.

```
 Windows Server (Dell R450, physical)
   └─ Hyper-V role 
        └─ Ubuntu Server VM  
             └─ Docker  
                  └─ Uptime Kuma container  
```
                  
## The install
When setting up on a Windows Server, first check whether Hyper-V is avalible on the host.

 -In powershell, run the following commands to check requirements. 
 ```
Get-WindowsFeature -Name Hyper-V
system info
```
<img width="796" height="93" alt="Screenshot_20260624_101630" src="https://github.com/user-attachments/assets/4795f914-3529-478f-bf0d-3d5f55e50014" />
`Get-WindowsFeature` shows whether the role is **Installed** or just
**Available**. In `systeminfo` scroll to the bottom of the **Hyper-V
Requirements** This section should show all four items as **Yes** (VM Monitor
Mode Extensions, Virtualization Enabled in Firmware, Second Level Address
Translation, Data Execution Prevention).
<img width="493" height="58" alt="Screenshot_20260624_101409" src="https://github.com/user-attachments/assets/43f66821-d9af-41ff-85a9-1dd13b147f4d" />
If Hyper-V shows as availble and all the requirements are checked, then the host is ready. Download your choice of server software, I will be using Ubuntu Server 20.04. Before installation I checked to see my NIC card. After location which port the NIC is using you need to create a external vswitch
```
Get-NetAdapter | Where-Object Status -eq 'Up'

# Bind the switch.
New-VMSwitch -Name "External-LAN" -NetAdapterName "<use-NIC-name>" -AllowManagementOS $true
```
After binding the vswitch you can check to make sure that it is active.

### Create Ubuntu Server VM.
In powershell run the following commands to start mounting and creating you VM in Hyper-V
```
New-VM -Name "UptimeKuma" -MemoryStartupBytes 2GB -Generation 2 `
  -NewVHDPath "C:\Hyper-V\UptimeKuma\UptimeKuma.vhdx" -NewVHDSizeBytes 20GB `
  -SwitchName "External-LAN"

Set-VM -Name "UptimeKuma" -ProcessorCount 2

# Attach the ISO
Add-VMDvdDrive -VMName "UptimeKuma" -Path "C:\<path-of-ISO"

# Gen 2 needs Secure Boot set for Linux (Microsoft UEFI CA template)
Set-VMFirmware -VMName "UptimeKuma" -SecureBootTemplate "MicrosoftUEFICertificateAuthority"

# Make sure it boots from the DVD first
$dvd = Get-VMDvdDrive -VMName "UptimeKuma"
Set-VMFirmware -VMName "UptimeKuma" -FirstBootDevice $dvd
```
After creating the VM, start it and open the console:

```powershell
Start-VM -Name "UptimeKuma"
vmconnect.exe localhost "UptimeKuma"
```

### Install Ubuntu Server

Work through the installer. The settings that matter:

- Set a **static IP** so the monitoring box never changes address. I used
  172.21.0.34/16, gateway 172.21.0.1.
- Check **Install OpenSSH server** so the VM can be managed remotely.
- Use the entire disk with LVM. Leave LUKS encryption **off** — a monitoring
  box needs to reboot on its own, and LUKS would stop boot to ask for a
  passphrase.
- Don't select any of the featured snaps. Docker gets installed separately.

One thing worth doing before installing: verify the ISO. I checked the
SHA256 of the download against Canonical's published checksum. My first
download didn't match, so I re-downloaded and re-checked before installing.

```powershell
Get-FileHash "C:\ISOs\ubuntu-24.04.4-live-server-amd64.iso" -Algorithm SHA256
```
<img width="1026" height="875" alt="image" src="https://github.com/user-attachments/assets/27aabb62-dc2b-4a02-a963-cbc165a9c9fa" />

### Install Docker and Uptime Kuma

After Ubuntu reboots, log in and update the system:

```bash
sudo apt update && sudo apt upgrade -y
```

Install Docker:
Install Docker with the following command and create a user name.
```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker cord
```

Reboot the VM so the group change takes effect, then run
Uptime Kuma:

```bash
docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:2
```

Kuma is now running. Reach the dashboard from any machine on the network at
`http://172.21.0.34:3001` and create the admin account on first load.

