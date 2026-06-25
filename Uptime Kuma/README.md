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
`Get-WindowsFeature` shows whether the role is **Installed** or just
**Available**. In `systeminfo` scroll to the bottom of the **Hyper-V
Requirements** This section should show all four items as **Yes** (VM Monitor
Mode Extensions, Virtualization Enabled in Firmware, Second Level Address
Translation, Data Execution Prevention).

If Hyper-V shows as availble and all the requirements are checked, then the host is ready. Download your choice of server software, I will be using Ubuntu Server 20.04. Before installation I checked to see my NIC card. After location which port the NIC is using you need to create a external vswitch
```
Get-NetAdapter | Where-Object Status -eq 'Up'

# Bind the switch.
New-VMSwitch -Name "External-LAN" -NetAdapterName "<use-NIC-name>" -AllowManagementOS $true
```
After binding the vswitch you can check to make sure that it is active.

### Create the VM.
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


