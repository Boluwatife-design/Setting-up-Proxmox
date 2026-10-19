# Getting started with Proxmox
- Proxmox was installed on a virtual machine ( used Oracle VM)
- Download the Proxmox VE ISO

      Get it from proxmox.com/en/downloads (Proxmox VE ISO installer)

- Create the VM in VirtualBox

      Type: Linux, Version: Debian (64-bit)
      RAM: 4096 MB minimum (8 GB if you can spare it)
      CPUs: 2 or more
      Disk: 32 GB+, dynamically allocated

     <img width="1659" height="841" alt="Screenshot 2026-10-19 155814" src="https://github.com/user-attachments/assets/b88d5602-4541-4f03-8e61-57396c248468" />

- Before starting it, go to Settings > System > Processor and tick Enable Nested VT-x/AMD-V. If it’s greyed out, close VirtualBox and run this in Command Prompt:

      "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "YourVMName" --nested-hw-virt on

  <img width="1877" height="916" alt="Screenshot 2026-10-19 160336" src="https://github.com/user-attachments/assets/0e52ba45-a243-4a13-b564-72d69e3ab6a1" />


- Network: set Adapter 1 to Bridged Adapter.

<img width="502" height="926" alt="Screenshot 2026-10-19 160505" src="https://github.com/user-attachments/assets/067719cf-595a-4f59-abb9-d1d52d5d9586" />

- Storage: attach the Proxmox ISO to the optical drive, start the VM, and follow the installer



