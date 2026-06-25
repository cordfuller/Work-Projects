# Uptime Kuma work set-up
This was a project that I setup to monitor are servers and devices if interuptions happen it can notify me. the way the stack will be built is.
 
 Windows Server (Dell R450, physical)
   └─ Hyper-V role 
        └─ Ubuntu Server VM  
             └─ Docker  
                  └─ Uptime Kuma container  
                  
### The install
When setting up on s windows server you need to check to see if Hyper-v is installed and avalible.

 -In powershell run the following commands to check requirments. 
- Get-WindowsFeature -Name Hyper-v
 - system info

At the bottom you should see if Hyper-V is ready for setup. After meeting all the requirments you can install your choice of OS for your VM, I will be using Ubuntu Server.
