## 2026-11-20 - Web server in a container, and reading its logs

**Goal:** Learn how a container that keeps running differs from one
that exits, and see what a web server records about requests.

**What I did:**
1. Ran nginx in the background (`docker run -d --name web nginx`) and
   compared its STATUS with hello-world's.
2. Used `docker logs`, `docker exec`, `docker stop` and `docker rm`,
   and watched `web` leave `docker ps` after stop, then leave
   `docker ps -a` after rm.
3. Re-ran it with `-p 127.0.0.1:8080:80`, sent requests with curl, and
   read the access and error logs.

**What happened:**
- `docker exec web nginx -v` printed `nginx/1.31.9`.
- While running, `web` showed STATUS "Up"; hello-world showed "Exited".
- `curl http://127.0.0.1:8080` returned the nginx welcome page and the
  access log recorded `GET / HTTP/1.1" 200 896`.
- `curl http://127.0.0.1:8080/does-not-exist` was logged as
  `404 153`, and the error log said nginx could not open
  `/usr/share/nginx/html/does-not-exist` (No such file or directory).
- The client address in both log lines was 172.17.0.1, not 127.0.0.1,
  although I ran curl from the VM itself.

**What I learned:**
- A web server is a program that waits for requests and answers them,
  so its container stays Up; hello-world exits because its program
  finishes.
- `-p 127.0.0.1:8080:80` publishes container port 80 on VM port 8080,
  reachable only from the VM itself.
- Scanners that match versions can report flaws that are already
  fixed, because vendors sometimes patch without changing the version
  number (backporting). Not yet observed in my lab; I will look for it
  when Grype, Trivy and Greenbone disagree.

**Open questions:**
- Is the client address 172.17.0.1 because Docker sits between the VM
  and the container? Hypothesis: yes. Test: curl the container's own
  address directly and compare the log line.

**Next:** Test the client-address hypothesis, then Day 2b: Docker
Compose, volumes and networks.


---


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

