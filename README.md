# Q670 NAS Hardware Setup and Power Optimization

<details>

<summary>Repo QR</summary>

<img src="./assets/repo-qr.png" width="250"/>

</details>

This repo is meant to detail the hardware specs and power optimization settings for my homelab NAS

## Hardware Spec

| | | | |
|---|---|---|---|
| CASE | Jonsbo N3 NAS Case | [[spec](https://www.jonsbo.com/en/products/N3.html)] [[amazon](https://www.amazon.com/dp/B0CMVBMVHT)] | |
| MOBO | CWWK (StoneStorm) Q670 8bay NAS Motherboard (White) | [[spec](https://www.cwwkipc.com/product/nas-storage/nas-motherboard/nas-b-q670-plus/)] [[amazon](https://www.amazon.com/dp/B0DP9JMD2Q)] | |
| CPU | Intel Core i5 14500 Processor | [[spec](https://www.intel.com/content/www/us/en/products/sku/236784/intel-core-i5-processor-14500-24m-cache-up-to-5-00-ghz/specifications.html)] [[amazon](https://www.amazon.com/dp/B0CQ27H8VY)] | |
| MEM | Corsair Vengance 96GB (2x 48gb) at 6000mhz (4800mhz actual) | [[spec](https://www.corsair.com/us/en/p/memory/cmk96gx5m2e6000z36/vengeance-96gb-2x48gb-ddr5-dram-6000mt-s-cl36-amd-expo-intel-xmp-memory-kit-cmk96gx5m2e6000z36#tab-techspecs)] [[amazon](https://www.amazon.com/dp/B0F7RY9V4N)] | Speed limited d/t CPU spec |
| PSU | Corsair SF750 | [[spec](https://www.corsair.com/us/en/p/psu/cp-9020186-na/sf-series-sf750-750-watt-80-plus-platinum-certified-high-performance-sfx-psu-cp-9020186-na)] | |
| NVME | Samsung PM981a | [[spec](https://www.techpowerup.com/ssd-specs/samsung-pm981a-256-gb.d784)] | |
| NVME | Adata XPG SX8200 | [[spec](https://www.techpowerup.com/ssd-specs/xpg-sx8200-pro-512-gb.d886)] | |
| HDD | 8x WD Easystore 8TB Schucked White Label | [[spec](https://www.westerndigital.com/products/portable-drives/wd-easystore-desktop-usb-3-0-hdd?sku=WDBAMA0080HBK-NESN)] | TODO: Watts when connected |

## Power Optimization

Using above hardware spec minus HDDs, following BIOS config is idling at 13-17 watts as measured by mains outlet power meter [[pic](./assets/power-meter.jpg)], with only ethernet connected to i226-LM LAN port. When display connected, power usage jumps to 21-25 watts.

Achieved CPU pkg at C8, P cores at C7, E cores at C6 in Powertop

### Notes

 - Power Optimization shamelessly compiled/stolen from R𝖾ddit discussion via R𝖾dlib [[here](https://safereddit.com/r/homelab/comments/1gmf67u/cwwk_q670_8bay_new_model_white/?limit=500)] [[archived](https://archive.ph/gKbbq)]
 - - There is a great wealth of useful info on this board in the nested in the discussion
 - Using second /u/Yonji1 bios "BIOS Q670-PLUS unlocked v7" [[link](https://safereddit.com/r/homelab/comments/1gmf67u/cwwk_q670_8bay_new_model_white/lylhek2/)]; archived in repo (repackaged to tar.gz)
 - - Also archived Stock BIOS and /u/Yonji1's first modified BIOS (repackaged to tar.gz)
 - Back 2 NVME slots 2 and 3 populated with Samsung and Adata NVMEs; slot 1 on front unpopulated
 - Enabled ASPM on all PCIe root ports. Will test disabling root port for NICS and x16 slot, etc. when have time *TODO: Update* 
 - - Enabled PCIe root port (3-4?) ASPM for both NICs without soft-locks or other issues with power gating disabled
 - Still need to figure out how to trace BIOS PCIe root port to the device it goes to. Experimentation? *TODO: Update*
 - Still need to take measurements with HDDs connected and idling, spun down *TODO: Update*

### In BIOS

 - Advanced > RC ACPI Settings > Native ASPM = Enabled
 - Advanced > Connectivity Configuration > CNVi Mode = Disable Integrated
 - Advanced > Connectivity Configuration > Discrete Bluetooth Interface = Disabled
 - Advanced > Power & Performance > CPU - Power Management Control > C States = Enabled
 - Advanced > Power & Performance > CPU - Power Management Control > Package C State Limit = C10
 - Advanced > PCH-FW Configuration > ME State = Disabled
 - Chipset > PCH-IO Configuration > PCI Express Configuration > PCH PCIE Power Gating = Disabled
 - Chipset > PCH-IO Configuration > PCI Express Configuration > PCI Express Root Port 1..20+ > ASPM = L1
 - Chipset > PCH-IO Configuration > PCI Express Configuration > PCI Express Root Port 1..20+ > L1 Substates = L1.1 and L1.2
 - Chipset > PCH-IO Configuration > HD Audio Configuration > HD Audio = Disabled
 - Chipset > System Agent (SA) Configuration > PCI Express Configuration > PCI Express Root Port 1..3 > ASPM = L1
 - Chipset > System Agent (SA) Configuration > DMI/OPI Configuration > DMI Gen3 ASPM = ASPM L1

### In Powertop

 - Enabled all tunables except lan ports
 - Powertop systemd service and tunables service in repo with used values
 - - Code taken and modified to suit current setup from [[here](https://github.com/EMH-Mark-I/Powertop-Systemd-auto-start-Deployment-Script)]

### Enable powersave cpu scaling governor

```
echo "powersave" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

### To check that aspm is enabled on PCI devices:

```
lspci -vv | awk '/ASPM/{print $0}' RS= | grep --color -P '(^[a-z0-9:.]+|ASPM )'
```
