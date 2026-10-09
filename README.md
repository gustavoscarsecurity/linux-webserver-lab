# Linux Network Security Lab
English | [Português](README.pt-BR.md)

Personal networking and security lab running on a Debian 13 VM. I configured UFW and accessed a web server on the VM from Windows to understand how services, ports and firewall rules work together.

## Environment
- **Host:** Windows 11.
- **VM:** Debian 13 on VirtualBox.
- **Network:** initially NAT; switched to bridged mode for the test.
- **Tools:** UFW, Python 3, `ip` and `ss`.

In bridged mode, the VM received the private address `192.168.18.247/24`. The `/24` is the network prefix, not a port. This address is specific to my setup and may change.

## What I did

### 1. Checked networking and services
```bash
ip a
ip route
sudo ss -tulpn
```

I checked addresses, routes and local TCP/UDP sockets. I identified Avahi in the UDP entries and CUPS on TCP port 631, bound to the loopback addresses `127.0.0.1` and `::1`.

The `ss` output shows local sockets; it does not prove that a service is reachable from another machine.

### 2. Configured the firewall
```bash
sudo apt update
sudo apt install ufw
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw status verbose
```

I confirmed that the firewall was active, with incoming traffic denied and outgoing traffic allowed by default. These policies apply to the VM. Replies to connections initiated by the VM are still allowed.

I also practiced adding and removing an exception:
```bash
sudo ufw allow 8080/tcp
sudo ufw delete allow 8080/tcp
```

### 3. Tested access from Windows
Inside a dedicated test directory, I started an HTTP server:
```bash
mkdir lab-web
cd lab-web
python3 -m http.server 8080
```

The `-m` option runs Python's `http.server` module, which serves the directory contents on port 8080.

I allowed the connection through UFW:
```bash
sudo ufw allow 8080/tcp
```

With the server running, I opened this address in the Windows browser:
```text
http://192.168.18.247:8080
```

The browser displayed **Directory listing for /**, confirming access to the server on the VM.

## Results and lessons learned
- Configured and checked UFW default policies.
- Added and removed a TCP/8080 rule.
- Successfully accessed the VM's web server from Windows.
- During the test, I stopped the server to run another command and had to start it again. Allowing a port through the firewall does not start or keep a service running.
