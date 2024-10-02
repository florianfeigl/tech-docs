+++
title = 'Flashing OpenWrt on your hardware'
date = 2024-09-30T20:16:51+02:00
draft = true 
tags = ["network", "software", "wifi", "ssh", "tftp"]
+++
# Flashing OpenWrt on a Ubiquiti AP AC Pro: A Journey from TFTP Frustration to SSH Success

![Ubiquiti AP AC Pro](/images/posts/uap-ac-pro-8.png)

*Image credit: https://techspecs.ui.com/unifi/wifi/uap-ac-pro#marketing-images*

## Introduction

As a network enthusiast, I've always been keen on customizing my devices to unlock their full potential. Recently, I decided to flash OpenWrt onto my **Ubiquiti AP AC Pro** to gain more control over its features. What seemed like a straightforward task turned into a learning experience that underscored the importance of digging deeper when faced with technical challenges.

*Specifications: https://techspecs.ui.com/unifi/wifi/uap-ac-pro#marketing-images*

## The Initial Plan: Flashing via TFTP

Flashing firmware via TFTP (Trivial File Transfer Protocol) is a common method recommended for devices like the Ubiquiti AP AC Pro. Confidently, I gathered all the necessary tools:

- **OpenWrt firmware** compatible with my device
- A **TFTP server** set up on my Linux machine
- An Ethernet cable to connect my computer directly to the access point

Following the standard procedure, I:

1. **Set up my PC's IP address** to be in the same subnet as the AP.
2. **Configured the TFTP server** to serve the OpenWrt firmware.
3. **Put the AP into TFTP recovery mode**, which involves pressing the reset button while powering on the device.
4. **Monitored the TFTP transfer** using `tcpdump` to ensure the firmware was being sent.

## Hitting a Wall: The TFTP Failure

Despite meticulous preparation, the TFTP method failed to deliver the desired result. The firmware transfer appeared to occur successfully—`tcpdump` confirmed data was being sent over UDP port 69. However, after the AP rebooted, it was still running the stock Ubiquiti firmware. There were no error messages or obvious signs pointing to the cause of the failure.

Attempts to access the OpenWrt interface at `192.168.1.1` proved futile. Instead, the device remained accessible at its default IP, `192.168.1.20`, and continued to exhibit Ubiquiti-specific behavior. Even SSHing into the device using `ubnt@192.168.1.20` worked, confirming that the stock firmware was intact.

## The Turning Point: Reading Deeper

Frustration set in. I had followed the TFTP flashing instructions to the letter, yet the AP stubbornly refused to accept the new firmware. It was time to dig deeper.

A bit of research led me to the [OpenWrt wiki page for the Ubiquiti UniFi AC series](https://openwrt.org/toh/ubiquiti/unifiac). Buried within the documentation, I discovered an alternative method that didn't rely on TFTP:

### The Non-Invasive SSH and `dd` Method

This approach leverages SSH access to the device and uses the `dd` command to write the firmware directly to the device's memory partitions. Here's a condensed version of the steps:

1. **Obtain the correct OpenWrt firmware** for the Ubiquiti AP AC Pro.
2. **Connect the AP directly to the PC** using an Ethernet cable. If using PoE, ensure only the PoE injector is in between.
3. **Set a static IP** on the PC (e.g., `192.168.1.10`) in the same subnet as the AP.
4. **SSH into the AP** using `ssh ubnt@192.168.1.20` (password: `ubnt`). If SSH fails, perform a hard reset.
5. **Transfer the firmware to the AP**:
   - If `scp` works:
     ```bash
     scp -O openwrt.bin ubnt@192.168.1.20:/tmp/sysupgrade.bin
     ```
   - If `scp` fails:
     ```bash
     cat openwrt.bin | ssh ubnt@192.168.1.20 "cat > /tmp/sysupgrade.bin"
     ```
6. **Unlock the memory partitions** (if necessary):
   ```bash
   echo "5edfacbf" > /proc/ubnthal/.uf
   ```
7. **Identify the correct memory partitions** by running `cat /proc/mtd`.
8. **Write the firmware to the partitions**:
   ```bash
   dd if=/tmp/sysupgrade.bin of=/dev/mtdblock2
   dd if=/tmp/sysupgrade.bin of=/dev/mtdblock3
   ```
9. **Set the boot flag** to boot from the new firmware:
   ```bash
   dd if=/dev/zero bs=1 count=1 of=/dev/mtdblock4
   ```
10. **Reboot the device**:
    ```bash
    reboot
    ```
11. **Access OpenWrt** at `http://192.168.1.1`.

## Success at Last

Following these steps, I was able to successfully flash OpenWrt onto my Ubiquiti AP AC Pro. The device rebooted, and I was greeted with the OpenWrt web interface—a welcome sight after hours of troubleshooting.

## Lessons Learned

This experience reinforced several important lessons:

- **Persistence Pays Off**: When the standard methods fail, it's crucial not to give up. There's often more than one way to achieve a goal.
- **Read the Documentation Thoroughly**: The answer was available in the documentation, but it required careful reading and attention to detail.
- **Understand Your Device**: Knowing that I could SSH into the AP opened up alternative avenues for flashing the firmware.
- **Community Resources Are Invaluable**: Forums, wikis, and user guides provided by the community can offer solutions not found in official manuals.

## Conclusion

Flashing custom firmware can be a challenging but rewarding endeavor. While the TFTP method is commonly recommended, it's not always the silver bullet. By exploring alternative methods and deepening your understanding of the device, you can overcome obstacles and achieve your objectives.

If you're attempting to flash OpenWrt onto a Ubiquiti AP AC Pro and encounter issues with TFTP, consider using the SSH and `dd` method. Just remember to proceed with caution—writing directly to memory partitions carries risks, and it's essential to follow the steps carefully.

---

**Disclaimer**: Modifying your device's firmware can void warranties and, if done incorrectly, may render the device unusable. Proceed at your own risk.
