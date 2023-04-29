# OpenWrt NXP pcie9098 driver for Mochabin

This is a driver built to support the NXP pcie9098 card that comes with the Golbalscale Mochabin.  

## Install:

```
opkg install http://github.com/Ahrotahn/openwrt-nxp-pcie9098/releases/download/230128/kmod-nxp-pcie9098_v22.03.4.ipk
```
```
reboot
```
Change the release and version in the link for the version of OpenWrt you have installed.<br>
You can see what versions are available on the [Releases page](https://github.com/Ahrotahn/openwrt-nxp-pcie9098/releases).

<br>

## Note:

* The Sep 05 2022 build of uboot does not reset the pcie device on reboot.  A poweroff may be required should the card end up in a bad state.  

* Both radios are capable of running in AC+ mode, but not at the same time.  

* STA/client mode will need `drv_mode=1` added to the section for the radio(s) in the configuration:
   <details><summary>/lib/firmware/nxp/wifi_mod_para.conf</summary>

   ```
   PCIE9098_0 = {
       drv_mode=1
       cfg80211_wext=0xf
       max_vir_bss=1
       cal_data_cfg=none
       ps_mode=2
       auto_ds=2
       host_mlme=1
       fw_name=nxp/pcieuart9098_combo_v1.bin
   }
   ```
   </details>

<br>

## Known working configuration:

### radio0:

Mode|Band |Channel      |Width 
----|-----|-------------|------
AX  |5 GHz|36 (5180 Mhz)|40 MHz

---

### radio1:

Mode|Band   |Channel     |Width 
----|-------|------------|------
N   |2.4 GHz|6 (2437 Mhz)|40 MHz

Allow legacy 802.11b rates `☑`

Set `Country Code` under the `Advanced Settings` tab.  

---

<details>
<summary>Stock Settings:</summary>

### radio0:

Mode|Channel      |Width 
----|-------------|------
AC  |36 (5180 Mhz)|40 MHz

---

### radio1:

Mode|Band   |Channel     |Width 
----|-------|------------|------
N   |2.4 GHz|6 (2437 Mhz)|40 MHz

Allow legacy 802.11b rates `☑`

Set `Country Code` under the `Advanced Settings` tab.  

---

</details>

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

<br>

## Logging Info:

To increase log verbosity edit `/etc/modules.d/nxp-pcie9098` and change `drvdbg=0x6` to `drvdbg=0x7`.  

For full debug info use `drvdbg=0x20037`.  
