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
When setting up on a Windows Server, first check whether Hyper-v is avalible on the host.

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

If Hyper-V shows as avalible and all the requirments are checked, then the host is ready. Run the following command.


You can install your choice of OS for your VM, I will be using Ubuntu Server.
