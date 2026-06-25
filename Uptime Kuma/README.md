# Uptime Kuma work set-up
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
At the bottom you should see if Hyper-V is ready for setup. After meeting all the requirments you can install your choice of OS for your VM, I will be using Ubuntu Server.
