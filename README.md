# OpenWrt NXP pcie9098 driver for Mochabin

This is a work-in-progress driver built to support the NXP pcie9098 card that comes with the Golbalscale Mochabin.  

## Install

```
opkg install http://github.com/Ahrotahn/openwrt-nxp-pcie9098/releases/download/v22.03.3/kmod-nxp-pcie9098_v22.03.3.ipk
```

<br>

## ⚠ IMPORTANT ⚠

* The Sep 05 2022 build of uboot does not reset the pcie device on reboot.  You will need to power off the device before the driver will work.  I'm still investigating whether this can be worked around in the driver, but FLRs aren't enough so this might require an update to uboot to resolve.  

* The hardware supports 802.11ax but the currently available firmware does not!  This includes the original firmware released with the Mochabin.  

<br>

## Known working configuration:

### radio0:

Mode|Channel      |Width 
----|-------------|------
AC  |36 (5180 Mhz)|40 MHz

---

### radio1:

Mode|Band   |Channel     |Width 
----|-------|------------|------
N   |2.4 GHz|6 (2437 Mhz)|40 MHz

`Allow legacy 802.11b rates ☑`

Set `Country Code` under the `Advanced Settings` tab.  

---

<details>
<summary>Example <code>/etc/config/wireless</code></summary>

```
config wifi-device 'radio0'
	option type 'mac80211'
	option cell_density '0'
	option band '5g'
	option country 'US'
	option channel '36'
	option htmode 'VHT40'

config wifi-device 'radio1'
	option type 'mac80211'
	option cell_density '0'
	option htmode 'HT40'
	option band '2g'
	option channel '6'
	option country 'US'
	option legacy_rates '1'

config wifi-iface 'wifinet0'
	option device 'radio0'
	option mode 'ap'
	option ssid 'YourAP1'
	option key 'password'
	option network 'lan'
	option encryption 'psk2'

config wifi-iface 'wifinet1'
	option device 'radio1'
	option mode 'ap'
	option ssid 'YourAP2'
	option encryption 'psk2'
	option key 'password'
	option network 'lan'
```

</details>

## Logging Info

To increase log verbosity edit `/etc/modules.d/nxp-pcie9098` and change `drvdbg=0x6` to `drvdbg=0x7`.  

To get full debug info use `drvdbg=0x20037`.  
