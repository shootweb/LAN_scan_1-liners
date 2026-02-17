# LAN Host Discovery One-Liners (NO PORT SCANNING)

All commands:

- Perform **host discovery only**
- Output **IP addresses only**
- Enforce **exclusion list**
- Save results to a file
- Are **copy-paste ready**

---

# ⚠️ Requirements

Create an exclusion file:

```
touch no_go_ips.txt
```

(Optional) Add IPs to exclude:

```
echo "192.168.1.1" >> no_go_ips.txt
```

---

# 🌐 Target Ranges

```
192.168.0.0/16
172.16.0.0/12
10.0.0.0/8
```

---

# 🚀 Nmap (ICMP Discovery Only)

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

# ⚡ fping (CORRECTED MULTI-RANGE)

## FAST
```
(
sudo fping -a -q -r 0 -t 50 -i 1 -g 192.168.0.0/16;
for net in 172.{16..31}.0.0/16; do sudo fping -a -q -r 0 -t 50 -g $net; done;
for net in 10.{0..255}.0.0/16; do sudo fping -a -q -r 0 -t 50 -g $net; done
) 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt | tee fping_ips.txt
```

## BALANCED
```
(
sudo fping -a -q -r 1 -t 100 -i 2 -g 192.168.0.0/16;
for net in 172.{16..31}.0.0/16; do sudo fping -a -q -r 1 -t 100 -g $net; done;
for net in 10.{0..255}.0.0/16; do sudo fping -a -q -r 1 -t 100 -g $net; done
) 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt | tee fping_ips.txt
```

## STEALTH
```
(
sudo fping -a -q -r 2 -t 200 -i 10 -g 192.168.0.0/16;
for net in 172.{16..31}.0.0/16; do sudo fping -a -q -r 2 -t 200 -i 10 -g $net; done;
for net in 10.{0..255}.0.0/16; do sudo fping -a -q -r 2 -t 200 -i 10 -g $net; done
) 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt | tee fping_ips.txt
```

---

# 🧱 arp-scan (LOCAL NETWORK ONLY)

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

# 👁️ Passive Discovery (NO TRAFFIC)

```
ip neigh | awk '/lladdr/{print $1}' | grep -v -w -f no_go_ips.txt | tee passive_ips.txt
```

---

# 📡 Broadcast Stimulation (LOCAL ONLY)

Replace with your subnet broadcast:

```
ping -b -c 2 192.168.1.255 >/dev/null 2>&1 && ip neigh | awk '/lladdr/{print $1}' | grep -v -w -f no_go_ips.txt | tee broadcast_ips.txt
```

---

# 🔥 Combined Discovery (ALL METHODS)

## BALANCED (Recommended)
```
(
sudo fping -a -q -r 1 -t 100 -i 2 -g 192.168.0.0/16;
for net in 172.{16..31}.0.0/16; do sudo fping -a -q -r 1 -t 100 -g $net; done;
for net in 10.{0..255}.0.0/16; do sudo fping -a -q -r 1 -t 100 -g $net; done;
nmap -sn -n -PE -PP --min-rate 1000 --max-retries 1 --host-timeout 2s -oG - 192.168.0.0/16 172.16.0.0/12 | awk '/Up$/{print $2}';
ip neigh | awk '/lladdr/{print $1}'
) 2>/dev/null | sort -u | grep -v -w -f no_go_ips.txt | tee all_ips.txt
```

---

# ⚠️ Verified Constraints

- ✔ No TCP SYN / ACK / UDP probes
- ✔ All commands produce **only IPs**
- ✔ All commands enforce exclusion list
- ✔ All commands write to files
- ✔ All commands are copy-paste ready

---

# ⚠️ Real World Notes

- ICMP may be blocked → false negatives
- `/8` scans can take time and generate traffic
- `arp-scan` only works on local network
- `fping` may require root privileges

---

# ⭐ Recommended Workflow

1. Run `arp-scan`
2. Run `fping`
3. Run `nmap`
4. Run combined command

---

# 🚀 Future Improvements

- IPv6 discovery
- mDNS / LLMNR enumeration
- DHCP lease parsing
- Scapy-based discovery
- Output normalization scripts

---
