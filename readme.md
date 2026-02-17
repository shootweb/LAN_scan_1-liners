# LAN Host Discovery One-Liners (No Port Probing)

All commands:

- Perform **host discovery only**
- Output **IP addresses only**
- Enforce **exclusion list**
- Save results to a file
- Are **copy-paste ready**

---

# 📁 Required File

Create your exclusion list:

```
no_go_ips.txt
```

Example:

```
192.168.1.1
10.0.0.5
172.16.1.10
```

---

# 📤 Output Files

Each command writes to a file:

| Tool       | Output File        |
|-----------|-------------------|
| Nmap      | nmap_ips.txt      |
| fping     | fping_ips.txt     |
| arp-scan  | arp_ips.txt       |
| passive   | passive_ips.txt   |
| combined  | all_ips.txt       |

---

# 🌐 Target Ranges

```
192.168.0.0/16
10.0.0.0/8
172.16.0.0/12
```

---

# 🚀 Nmap (ICMP Only)

## FAST
```
nmap -sn -n -PE --min-rate 2000 --max-retries 0 --host-timeout 1s --exclude-file no_go_ips.txt -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}' | tee nmap_ips.txt
```

## BALANCED
```
nmap -sn -n -PE -PP --min-rate 1000 --max-retries 1 --host-timeout 2s --exclude-file no_go_ips.txt -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}' | tee nmap_ips.txt
```

## STEALTH
```
nmap -sn -n -PE -PP --min-rate 100 --max-retries 2 --host-timeout 5s --exclude-file no_go_ips.txt -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}' | tee nmap_ips.txt
```

---

# ⚡ fping (Large Range)

## FAST
```
fping -a -q -r 0 -t 50 -i 1 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null | grep -v -w -f no_go_ips.txt | tee fping_ips.txt
```

## BALANCED
```
fping -a -q -r 1 -t 100 -i 2 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null | grep -v -w -f no_go_ips.txt | tee fping_ips.txt
```

## STEALTH
```
fping -a -q -r 2 -t 200 -i 10 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null | grep -v -w -f no_go_ips.txt | tee fping_ips.txt
```

---

# ⚡ fping (10.0.0.0/8 Chunked)

## FAST
```
for net in 10.{0..255}.0.0/16; do fping -a -q -r 0 -t 50 -g $net; done 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt | tee fping_10_ips.txt
```

## BALANCED
```
for net in 10.{0..255}.0.0/16; do fping -a -q -r 1 -t 100 -g $net; done 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt | tee fping_10_ips.txt
```

## STEALTH
```
for net in 10.{0..255}.0.0/16; do fping -a -q -r 2 -t 200 -i 10 -g $net; done 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt | tee fping_10_ips.txt
```

---

# 🧱 arp-scan (Local Network Only)

## FAST
```
sudo arp-scan --localnet --quiet | awk '/^[0-9]+\./{print $1}' | grep -v -w -f no_go_ips.txt | tee arp_ips.txt
```

## BALANCED
```
sudo arp-scan --localnet --retry=2 --timeout=200 --quiet | awk '/^[0-9]+\./{print $1}' | grep -v -w -f no_go_ips.txt | tee arp_ips.txt
```

## STEALTH
```
sudo arp-scan --localnet --retry=3 --timeout=500 --quiet | awk '/^[0-9]+\./{print $1}' | grep -v -w -f no_go_ips.txt | tee arp_ips.txt
```

---

# 👁️ Passive Discovery

```
ip neigh | awk '/lladdr/{print $1}' | grep -v -w -f no_go_ips.txt | tee passive_ips.txt
```

---

# 📡 Broadcast Stimulation

```
ping -b -c 2 192.168.1.255 >/dev/null 2>&1 && ip neigh | awk '/lladdr/{print $1}' | grep -v -w -f no_go_ips.txt | tee broadcast_ips.txt
```

---

# 🔥 Combined Discovery

## FAST
```
(
fping -a -q -r 0 -t 50 -i 1 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null;
for net in 10.{0..255}.0.0/16; do fping -a -q -r 0 -t 50 -g $net; done 2>/dev/null;
nmap -sn -n -PE --min-rate 2000 --max-retries 0 --host-timeout 1s -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}';
ip neigh | awk '/lladdr/{print $1}'
) | sort -u | grep -v -w -f no_go_ips.txt | tee all_ips.txt
```

## BALANCED
```
(
fping -a -q -r 1 -t 100 -i 2 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null;
for net in 10.{0..255}.0.0/16; do fping -a -q -r 1 -t 100 -g $net; done 2>/dev/null;
nmap -sn -n -PE -PP --min-rate 1000 --max-retries 1 --host-timeout 2s -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}';
ip neigh | awk '/lladdr/{print $1}'
) | sort -u | grep -v -w -f no_go_ips.txt | tee all_ips.txt
```

## STEALTH
```
(
fping -a -q -r 2 -t 200 -i 10 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null;
for net in 10.{0..255}.0.0/16; do fping -a -q -r 2 -t 200 -i 10 -g $net; done 2>/dev/null;
nmap -sn -n -PE -PP --min-rate 100 --max-retries 2 --host-timeout 5s -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}';
ip neigh | awk '/lladdr/{print $1}'
) | sort -u | grep -v -w -f no_go_ips.txt | tee all_ips.txt
```

---

# ⚠️ Notes

- `arp-scan` requires root privileges
- ICMP may be blocked → expect false negatives
- `/8` scans can take time and generate noise
- Always ensure authorization before scanning

---

# ⭐ Summary

- **fping** = fastest for large networks
- **arp-scan** = most reliable on LAN
- **nmap** = additional coverage
- **passive** = zero-noise discovery
- **combined** = best overall results

---
