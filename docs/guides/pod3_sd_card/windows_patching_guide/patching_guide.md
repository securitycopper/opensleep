# Pod 3 SD Card Patching Guide - Windows

This guide provides Windows-specific instructions for patching Pod 3 SD card images to enable SSH access and install OpenSleep.

## ⚠️ CRITICAL WARNING ⚠️

**DO NOT FORMAT THE SD CARD WHEN WINDOWS PROMPTS YOU!**

When you insert the SD card into your Windows computer, Windows may show a popup saying:
- "You need to format the disk in drive X: before you can use it"
- "The disk is not formatted"
- "Format disk now?"

**CLICK "CANCEL" OR CLOSE THE DIALOG!**

🛑 **DO NOT CLICK FORMAT!** 🛑

The SD card contains a Linux filesystem (ext4) that Windows cannot read. **The data IS there**, Windows just doesn't understand the Linux file system. If you format the disk, **all your data will be permanently lost** and you'll need to start over from scratch.

---

## Overview

At a high level, here is the process:
1. Create an image backup of the SD card
2. Copy the image to Ubuntu running in WSL
3. Run the patching script to modify the image
4. Write the modified image back to the SD card 


## Prerequisites

### Required Software

- **Windows Subsystem for Linux (WSL2)** with Ubuntu distribution
- **PowerShell** - Built into Windows
- **Win32 Disk Imager** - [Download here](https://win32diskimager.org/) for reading/writing SD card images

### Required Hardware

- **Micro SD Card Reader** - The author used [TS-RDF5K model](https://www.amazon.com/dp/B009D79VH4)
- **Your Pod 3 SD card**

### Optional Tools

- **ImageUSB by PassMark** - [Download here](https://www.osforensics.com/tools/write-usb-images.html) - Can create backups but not recommended for patching

### Installing WSL2

1. Open PowerShell as Administrator
2. Run the following command:
   ```powershell
   wsl --install
   ```
3. Restart your computer when prompted
4. Complete the Ubuntu setup when it launches
5. Create a username and password for your Ubuntu instance

---

## Step-by-Step Instructions

### Step 1: Create SD Card Image

1. Insert your Pod 3 SD card into the card reader
2. Open **Win32 Disk Imager**
3. Select the SD card device (verify it's the correct drive!)
4. Choose a location and filename for the image (e.g., `pod3_original.img`)
5. Click **Read** to create the image
6. Wait for the process to complete

**Note:** Win32 Disk Imager will read the entire SD card even if you select the second partition.

### Step 2: Gather Required Files

Before patching, collect these files:
- `pod3_original.img` - Image file from Step 1
- Your SSH public key (e.g., `~/.ssh/id_rsa.pub` or `~/.ssh/id_ed25519.pub`)
- `opensleep` - The opensleep binary
- `opensleep.service` - The systemd service file
- `config.ron` - Your opensleep configuration file
- `full_patch_workflow.sh` - The patching script from `scripts/`

### Step 3: Run the Patching Script

1. Open WSL (search for "Ubuntu" in Windows Start menu)
2. Navigate to where your files are located (Windows drives are mounted at `/mnt/`, e.g., `/mnt/c/Users/YourName/`)
3. Make the script executable:
   ```bash
   chmod +x ./full_patch_workflow.sh
   ```
4. Run the patching script with your arguments:

**Basic example:**
```bash
./full_patch_workflow.sh \
  -i pod3_original.img \
  -k ~/.ssh/id_rsa.pub \
  -s "YourWiFiSSID" \
  -p "YourWiFiPassword" \
  -b ./opensleep \
  -S ./opensleep.service \
  -c ./config.ron \
  -d
```

**Real-world example (with file paths):**
```bash
/mnt/f/workspaces/opensleep/scripts/full_patch_workflow.sh \
  -i "/mnt/f/drive images/pod3_original.img" \
  -k /mnt/c/Users/YourName/.ssh/id_ed25519.pub \
  -s "MyWiFiNetwork" \
  -p "MyWiFiPassword" \
  -b "/mnt/f/drive images/opensleep" \
  -S "/mnt/f/drive images/opensleep.service" \
  -c "/mnt/f/drive images/config.ron" \
  -d
```

**Script arguments:**
- `-i` - Input image file (required)
- `-k` - SSH public key file (required)
- `-s` - WiFi SSID (optional, but recommended)
- `-p` - WiFi password (optional, but recommended)
- `-b` - opensleep binary (optional)
- `-S` - opensleep.service file (optional)
- `-c` - config.ron file (optional)
- `-d` - Disable Eight Sleep services (optional, use with opensleep)

**⚠️ Note about `-d` flag:**
Using `-d` disables Eight Sleep services and prevents normal Eight Sleep app pairing. To restore Eight Sleep functionality, you must reflash the original image.

The script will create a patched image named `pod3_original-patched.img` in the same directory as the input image.

### Step 4: Write Patched Image to SD Card

1. Insert your SD card into the card reader
2. Open **Win32 Disk Imager**
3. Select your patched image file (e.g., `pod3_original-patched.img`)
4. Select the SD card device
5. **⚠️ VERIFY IT'S THE CORRECT DRIVE!** 
   - Double-check by looking at "Safely Remove USB" in the taskbar
   - It should show 3 partitions for your SD card
6. Click **Write** to write the image
7. Wait for the process to complete
8. Safely eject the SD card

### Step 5: Factory Reset the Pod

1. Insert the SD card back into your Pod 3
2. Hold down the small button on the Pod Hub while powering it on
3. Keep holding until the factory reset process begins
4. The Pod will boot with your patched image

---

## Verification

After the Pod boots up:

1. **SSH Access:**
   ```bash
   ssh rewt@<pod-ip-address> -p 8822
   ```
   
2. **Check opensleep status** (if installed):
   ```bash
   systemctl status opensleep
   ```

3. **Check WiFi connection:**
   ```bash
   nmcli connection show
   ```

---

## Troubleshooting

### SD card not detected
- Try a different USB port
- Ensure the card reader is compatible with your SD card size

### Image too large
- The patched image should be approximately the same size as the original
- Ensure you have enough free space (requires ~16GB during patching)

### SSH connection fails
- Verify the Pod is on your network: `ping <pod-ip>`
- Check SSH port: `8822` (not the standard `22`)
- Ensure your public key was added correctly

### WiFi not connecting
- Verify SSID and password are correct
- Check router compatibility (2.4GHz network recommended)
- Look at Pod logs after SSH access: `journalctl -u ssh-early.service`

---

## Additional Resources

- [Main SETUP.md](../../../../SETUP.md) - General setup instructions
- [OpenSleep README](../../../../README.md) - Project overview
- See the `scripts/full_patch_workflow.sh` for detailed script options (`-h` flag) 