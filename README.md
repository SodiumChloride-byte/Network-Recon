# Network-Recon
Nmap and Wireshark analysis

# Task 1: Network Reconnaissance with Nmap

## Objective
The goal of this task is to learn basic network reconnaissance by scanning my local network for open ports. This helps understand how devices expose services and what risks can come from them.

---

## Steps I Followed

1. **Installed Nmap**
   - Downloaded and installed Nmap from the official site: [https://nmap.org/download.html](https://nmap.org/download.html).

2. **Found my local IP range**
   - Checked my IP using:
     - On Windows: `ipconfig`
     - On Linux/Mac: `ifconfig` or `ip addr show`
   - My local IP range: `192.168.33.128/24`.

3. **Ran an Nmap TCP SYN scan**
   ```bash
   nmap -sS 192.168.33.128/24
