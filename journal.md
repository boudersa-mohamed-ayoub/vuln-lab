\## 2026-11-19 - Host baseline, Targets VM and Docker



\*\*Goal:\*\* Confirm the laptop can run the lab safely, then build the

`targets` VM with Docker installed.



\*\*Rule #1:\*\* Scanners only ever touch the VirtualBox host-only network,

never my home Wi-Fi (192.168.100.0/24).



\*\*Host baseline (checked before installing anything):\*\*

\- Lenovo ThinkPad X1 Carbon Gen 10, i7-1260P, 32 GB RAM, 1 TB NVMe

\- ReviOS 11 (Windows 11 Pro, build 26200)

\- CPU virtualization enabled in firmware; Hyper-V, VirtualMachinePlatform

&#x20; and HypervisorPlatform all Disabled (VirtualBox gets the CPU features)

\- Defender real-time protection on; firewall on for all three profiles

\- WSL not installed (deliberately left off, as it would enable the

&#x20; Windows hypervisor)



\*\*What I did:\*\*

1\. Used the built-in VirtualBox host-only network (192.168.56.1/24).

2\. Created VM `targets` (Ubuntu Server 24.04 LTS, 4 GB RAM, 2 vCPUs,

&#x20;  40 GB thin disk). Adapter 1 = NAT, Adapter 2 = host-only.

3\. Connected from PowerShell with SSH and installed Docker Engine from

&#x20;  Docker's official apt repository.

4\. Ran `sudo docker run hello-world` to test.



\*\*What happened:\*\*

\- `ip a` in the VM showed lo 127.0.0.1/8, enp0s3 10.0.2.15/24 (NAT) and

&#x20; enp0s8 192.168.56.101/24 (host-only).

\- First PowerShell was not elevated, so `Get-WindowsOptionalFeature`

&#x20; failed with "The requested operation requires elevation". Re-ran as

&#x20; administrator.

\- Could not copy text out of the VM console window; SSH solved it.

\- `hello-world` pulled from Docker Hub, printed its message, and exited.



\*\*What I learned:\*\*

\- An image is a recipe; a container is a running copy of it. The

&#x20; hello-world container stopped because its only program finished.

\- An adapter is the plug on a machine; a network is what it plugs into.

\- Bridged mode would put a vulnerable VM on the same network as real

&#x20; devices, which is why the lab never uses it.

\- A /24 netmask means the first 24 bits identify the network and the last 8 identifies the machine and are given by the DHCP server to each machine on the same network.



\*\*Open questions:\*\*

\- What exactly does each flag in the Docker install commands do? What does -fsSL do in the curl command?



\*\*Next:\*\* Snapshot `clean-docker-installed`, commit this repo, then

Docker Compose.

