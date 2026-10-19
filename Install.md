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


# Start the VM and Install

- Click Ok to save your settings
- Select your Proxmox VM in the list and click Start. A window opens and boots from the ISO.
- At the blue Proxmox menu, choose Install Proxmox VE (Graphical) and press Enter.
<img width="1013" height="756" alt="image" src="https://github.com/user-attachments/assets/05f80aba-8f44-4de0-a814-e03a686c3598" />

## Error
<img width="1327" height="784" alt="image" src="https://github.com/user-attachments/assets/45b157df-fa99-4484-9d38-8e97b1a1c666" />
- Press Win+R, type optionalfeatures, and untick: Hyper-V, Virtual Machine Platform, Windows Hypervisor Platform, and Windows Subsystem for Linux (if you use it).
<img width="868" height="834" alt="image" src="https://github.com/user-attachments/assets/44842bfe-2879-4f5e-880b-f6e7e30cadd2" />
- Open Windows Security > Device security > Core isolation details and turn Memory integrity OFF.
- Open Command Prompt as Administrator and run: bcdedit /set hypervisorlaunchtype off
- Restart Windows.





