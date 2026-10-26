# What an LXC container is: a lightweight Linux system that shares Proxmox’s kernel. It starts in seconds and uses very little RAM, and it doesn’t need KVM, so it works in your nested setup

## Step 1: Download a template
- In the left tree, click local (pve).
- Click CT Templates, then Templates.
- Search for debian-13 (or debian-12 if 13 isn’t listed), select it, and click Download.
- Wait for the task log to show TASK OK, then close it.
<img width="1858" height="924" alt="image" src="https://github.com/user-attachments/assets/ee610bc0-41bf-4dad-813f-aeaff25f3ed9" />
<img width="1757" height="805" alt="image" src="https://github.com/user-attachments/assets/b3aaaf0a-6056-4047-a71b-62cdbec9566d" />

## Step 2: Create the container
- Click Create CT (top right, blue button) and fill in the tabs:

      Tab	Setting
      General	Hostname: ct-test. Set a root password. Leave “Unprivileged container” ticked.
      Template	Storage: local. Template: the Debian one you downloaded.
      Disks	Storage: local-lvm. Size: 4 GB.
      CPU	Cores: 1
      Memory	Memory: 512 MB, Swap: 512 MB
      Network	Bridge: vmbr0. IPv4: Static, IP 192.168.56.20/24. Gateway: 192.168.56.10 (your Proxmox host).
      DNS	Set DNS server to 8.8.8.8
      Confirm	Tick Start after created, then Finish
      
### N/B: This is suited to your organization needs.

<img width="1911" height="885" alt="Screenshot 2026-10-26 152143" src="https://github.com/user-attachments/assets/e2d2a85b-c700-4e07-a885-9a8ecbe6f78f" /> <img width="759" height="566" alt="image" src="https://github.com/user-attachments/assets/6834e8cd-2301-4701-9bf2-56e281451bee" />

<img width="768" height="578" alt="image" src="https://github.com/user-attachments/assets/1e179a1b-d778-4ff1-83b2-9e62d35524e7" />

## Step 3: Open and explore it
- Click the container in the left tree, then Console.
- Log in as root with the password you set.
- Try these:
  
         hostname
         ip a
         free -m
         ping -c 3 192.168.56.10
  
- Back in the web UI, look at the Summary tab (CPU, memory, uptime) and the Resources, Network, and Options tabs.
### Note: At this point the container can reach Proxmox but not the internet, because the host-only network has no route out.




