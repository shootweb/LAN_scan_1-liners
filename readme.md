# LAN Host Discovery One-Liners (No Port Probing)

This repository contains **host discovery one-liners** designed for internal network reconnaissance **without any port scanning**.

All commands:
- Perform **host discovery only**
- Output **IP addresses only**
- Support an **exclusion list**
- Are optimized for **different speed modes**

---

# ⚠️ Disclaimer

- These commands generate network traffic.
- Only use them on networks you are **authorized to test**.
- Large scans (/8) can trigger IDS/IPS or impact network performance.

---

# 🧠 Methodology

This repo combines:

- **ICMP discovery** (Layer 3)
- **ARP discovery** (Layer 2)
- **Passive discovery** (no packets sent)

No techniques use:
- TCP SYN
- TCP ACK
- UDP port probing

---

# 📁 Exclusion List

Create a file named:

```
no_go_ips.txt
```

Example:

```
192.168.1.1
10.0.0.5
172.16.1.10
```

All commands use:

```
grep -v -w -f no_go_ips.txt
```

This ensures **exact matching**.

---

# 🌐 Target Ranges

Examples used:

```
192.168.0.0/16
10.0.0.0/8
172.16.0.0/12
```

---

# 🚀 Nmap (ICMP Discovery Only)

## FAST
```
nmap -sn -n -PE --min-rate 2000 --max-retries 0 --host-timeout 1s --exclude-file no_go_ips.txt -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}'
```

## BALANCED
```
nmap -sn -n -PE -PP --min-rate 1000 --max-retries 1 --host-timeout 2s --exclude-file no_go_ips.txt -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}'
```

## STEALTH
```
nmap -sn -n -PE -PP --min-rate 100 --max-retries 2 --host-timeout 5s --exclude-file no_go_ips.txt -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}'
```

### Notes
- Does NOT scale well to `/8`
- Prefer fping for very large networks

---

# ⚡ fping (Primary Tool for Large Networks)

## FAST
```
fping -a -q -r 0 -t 50 -i 1 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null | grep -v -w -f no_go_ips.txt
```

## BALANCED
```
fping -a -q -r 1 -t 100 -i 2 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null | grep -v -w -f no_go_ips.txt
```

## STEALTH
```
fping -a -q -r 2 -t 200 -i 10 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null | grep -v -w -f no_go_ips.txt
```

---

# ⚡ fping (Chunked /8 Scan)

Scanning `/8` directly is inefficient. Use chunking:

## FAST
```
for net in 10.{0..255}.0.0/16; do fping -a -q -r 0 -t 50 -g $net; done 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt
```

## BALANCED
```
for net in 10.{0..255}.0.0/16; do fping -a -q -r 1 -t 100 -g $net; done 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt
```

## STEALTH
```
for net in 10.{0..255}.0.0/16; do fping -a -q -r 2 -t 200 -i 10 -g $net; done 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt
```

---

# 🧱 arp-scan (Layer 2 Discovery - Local Network Only)

## FAST
```
sudo arp-scan --localnet --quiet | awk '/^[0-9]+\./{print $1}' | grep -v -w -f no_go_ips.txt
```

## BALANCED
```
sudo arp-scan --localnet --retry=2 --timeout=200 --quiet | awk '/^[0-9]+\./{print $1}' | grep -v -w -f no_go_ips.txt
```

## STEALTH
```
sudo arp-scan --localnet --retry=3 --timeout=500 --quiet | awk '/^[0-9]+\./{print $1}' | grep -v -w -f no_go_ips.txt
```

### Notes
- Only works on **local broadcast domain**
- Most reliable LAN discovery method

---

# 👁️ Passive Discovery (ARP Cache)

## FAST / BALANCED / STEALTH (same)
```
ip neigh | awk '/lladdr/{print $1}' | grep -v -w -f no_go_ips.txt
```

### Alternative
```
arp -an | awk -F '[()]' '{print $2}' | grep -E '^[0-9]+\.' | grep -v -w -f no_go_ips.txt
```

### Notes
- No network traffic
- Depends on previously seen hosts

---

# 📡 Broadcast Stimulation (Optional)

```
ping -b -c 2 192.168.1.255 >/dev/null 2>&1 && ip neigh | awk '/lladdr/{print $1}' | grep -v -w -f no_go_ips.txt
```

### Notes
- May trigger additional ARP entries
- Works only in local subnet
- Often blocked in modern networks

---

# 🔥 Combined Discovery (Recommended)

## FAST
```
(
fping -a -q -r 0 -t 50 -i 1 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null;
for net in 10.{0..255}.0.0/16; do fping -a -q -r 0 -t 50 -g $net; done 2>/dev/null;
nmap -sn -n -PE --min-rate 2000 --max-retries 0 --host-timeout 1s -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}';
ip neigh | awk '/lladdr/{print $1}'
) | sort -u | grep -v -w -f no_go_ips.txt
```

## BALANCED
```
(
fping -a -q -r 1 -t 100 -i 2 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null;
for net in 10.{0..255}.0.0/16; do fping -a -q -r 1 -t 100 -g $net; done 2>/dev/null;
nmap -sn -n -PE -PP --min-rate 1000 --max-retries 1 --host-timeout 2s -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}';
ip neigh | awk '/lladdr/{print $1}'
) | sort -u | grep -v -w -f no_go_ips.txt
```

## STEALTH
```
(
fping -a -q -r 2 -t 200 -i 10 -g 192.168.0.0/16 172.16.0.0/12 2>/dev/null;
for net in 10.{0..255}.0.0/16; do fping -a -q -r 2 -t 200 -i 10 -g $net; done 2>/dev/null;
nmap -sn -n -PE -PP --min-rate 100 --max-retries 2 --host-timeout 5s -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}';
ip neigh | awk '/lladdr/{print $1}'
) | sort -u | grep -v -w -f no_go_ips.txt
```

---

# ⚖️ Tradeoffs

| Mode     | Speed | Accuracy | Noise |
|----------|------|---------|------|
| FAST     | High | Lower   | High |
| BALANCED | Medium | Medium | Medium |
| STEALTH  | Low  | Higher  | Low |

---

# ⭐ Key Takeaways

- **fping is best for large ranges**
- **arp-scan is best for local LAN**
- **nmap improves coverage but is slower**
- **passive discovery is free visibility**
- Combining methods reduces false negatives

---

# 🚀 Future Improvements

- IPv6 discovery
- Scapy-based custom probes
- DHCP lease parsing
- mDNS / LLMNR discovery
- Visualization of discovered hosts

---
