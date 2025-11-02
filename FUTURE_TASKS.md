# Future Tasks and Enhancements

This document tracks potential improvements and features for the Pod 3 patching workflow and opensleep project.

## Priority: Medium

### 1. NTP Time Synchronization
**Status:** Planned  
**Description:** Configure NTP (Network Time Protocol) for accurate system time  
**Problem:** Currently the system clock shows incorrect time (e.g., "Wed 2021-03-24" when actual date is "Sun 2025-11-02")  
**Solution Ideas:**
- Add systemd-timesyncd configuration
- Configure NTP server (e.g., pool.ntp.org, time.google.com)
- Ensure service starts after network is online
- Consider adding `-t` flag to patching script for custom NTP server

**Implementation Notes:**
```bash
# Example systemd-timesyncd configuration
# File: /etc/systemd/timesyncd.conf
[Time]
NTP=time.google.com pool.ntp.org
FallbackNTP=time.cloudflare.com
```

---

### 2. SSH Key Cleanup
**Status:** Planned  
**Description:** Remove/disable Eight Sleep's SSH keys for improved security  
**Current Behavior:** Eight Sleep's authorized keys may still exist on the system  
**Solution Ideas:**
- Comment out Eight Sleep SSH keys in `/home/rewt/.ssh/authorized_keys`
- Completely remove their keys and only keep user-provided key
- Add `-r` flag (remove-eightsleep-keys) to patching script
- Document which keys belong to Eight Sleep vs user

**Security Benefit:**
- Ensures only your SSH key has access
- Prevents potential unauthorized access from Eight Sleep infrastructure

---

### 3. Static IP Address Configuration
**Status:** Planned  
**Description:** Option to set static IP address instead of DHCP  
**Use Case:** Easier to find Pod on network, stable IP for automation  
**Solution Ideas:**
- Add `-I` flag for static IP address (e.g., `-I 192.168.1.100`)
- Add `-G` flag for gateway (e.g., `-G 192.168.1.1`)
- Add `-N` flag for DNS server (e.g., `-N 8.8.8.8`)
- Configure NetworkManager connection with static IP
- Update `wpa_supplicant` or NetworkManager config

**Implementation Notes:**
```bash
# Example NetworkManager static IP config
# Could be added to /etc/NetworkManager/system-connections/wlan0
[ipv4]
method=manual
address1=192.168.1.100/24,192.168.1.1
dns=8.8.8.8;8.8.4.4;
```

**Alternative:** Use `nmcli` commands in a service to set static IP after connection

---

## Priority: Low

### 4. On-Device opensleep Compilation
**Status:** Research Needed  
**Description:** Build opensleep binary directly on the Pod 3 device  
**Current Workflow:** Cross-compile on development machine, copy to Pod  
**Potential Benefits:**
- No cross-compilation needed
- Always up-to-date with latest code
- Simplifies development workflow

**Challenges:**
- Pod 3 has limited resources (CPU, RAM, storage)
- Need to install Rust toolchain (~1-2GB)
- Compilation time may be very slow on ARM device
- May require SD card expansion for build dependencies

**Research Questions:**
1. How much free space is available on the Pod 3 rootfs?
2. Can the device handle Rust compilation? (Memory constraints)
3. What's the compilation time? (Could be 10+ minutes on ARM)
4. Is it worth the complexity vs cross-compilation?

**Alternative Approach:**
- Use GitHub Actions or CI/CD to build binaries
- Host pre-built binaries for easy download
- Keep cross-compilation as primary method

**Investigation Steps:**
```bash
# Check available disk space
df -h

# Check RAM
free -h

# Estimate Rust toolchain size
# Typically requires:
# - rustc: ~200MB
# - cargo: ~50MB  
# - Build cache: ~500MB-1GB
# - Dependencies: varies
```

---

## Completed Tasks

✅ **MAC Address Persistence** - Implemented and tested (November 2, 2025)  
✅ **WiFi Auto-Connect** - Implemented and tested  
✅ **Password Authentication** - Implemented and tested (November 2, 2025)  
✅ **SSH Key Authentication** - Implemented and tested  
✅ **Eight Sleep Service Disabling** - Implemented and tested  
✅ **Factory Reset Survival** - All configurations persist (tested)  
✅ **NetworkManager MAC Randomization Fix** - Critical fix implemented  

---

## Contributing

If you implement any of these features or have additional ideas, please:
1. Create a branch from `main` or current development branch
2. Update this document with your progress
3. Submit a pull request with detailed description
4. Update the "Tested Features" section in the patching guide

---

## Notes

- Priority levels are subjective and may change based on user needs
- Some features may require significant testing before merge
- Consider backwards compatibility when adding new flags to patching script
- Document all new features in the patching guide

Last Updated: November 2, 2025
