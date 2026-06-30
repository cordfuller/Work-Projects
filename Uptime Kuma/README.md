# Uptime Kuma Work set-up
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
systeminfo
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
```
Get-VMSwitch
```

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

```
Start-VM -Name "UptimeKuma"
vmconnect.exe localhost "UptimeKuma"
```
### Verify the ISO 
One thing worth doing before installing is verify the ISO. I checked the
SHA256 of the download against Canonical's published checksum. My first
download didn't match, during the install my download got corrupted, so I redownloaded and rechecked before installing and it passed.

```
Get-FileHash "C:\ISOs\ubuntu-24.04.4-live-server-amd64.iso" -Algorithm SHA256
```
<img width="1026" height="875" alt="image" src="https://github.com/user-attachments/assets/27aabb62-dc2b-4a02-a963-cbc165a9c9fa" />

### Install Ubuntu Server

Work through the installer. The settings that matter:

<img width="782" height="227" alt="image" src="https://github.com/user-attachments/assets/fd779b38-bd47-44ee-8a80-9e841d4c06c2" />

- Set a **static IP** so the VM never changes address. I used
  172.21.0.34/16, gateway 172.21.0.1.
- Check **Install OpenSSH server** so the VM can be managed remotely.
- Use the entire disk with LVM. 
- Don't select any of the featured snaps. Docker gets installed separately.
- 
<img width="1026" height="875" alt="image" src="https://github.com/user-attachments/assets/27aabb62-dc2b-4a02-a963-cbc165a9c9fa" />

### Install Docker and Uptime Kuma

After Ubuntu reboots, log in and update the system:

```
sudo apt update && sudo apt upgrade -y
```

Install Docker with the following command and create a user name.
```
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker cord
```

Reboot the VM so the group change takes effect, then run
Uptime Kuma:

```bash
docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:2
```

Kuma is now running. Reach the dashboard from any machine on the network at
`<ip>:3001` and create the admin account on first load.

<img width="1915" height="945" alt="Screenshot_20260626_143818" src="https://github.com/user-attachments/assets/c1575880-791d-4baa-b508-3fe5a089973d" />

# Wrap up
Uptime Kuma is an amazing tool that can help you monitor your work or home network. There are a ton of features for monitoring devices, database servers, port monitoring, DNS Records, and much more. You can also set up notifications for 70+ services.

The one real shortcoming is if your ISP goes down, the whole stack is sitting behind the dead connection, so it can't actually reach out to notify you. To counter that, I plan on renting a $5/month VPS service down the road to monitor our servers externally so that way the monitoring lives outside the network it's watching.

This was a fun project that I really enjoyed, and I'll be using it on my own home network too.
