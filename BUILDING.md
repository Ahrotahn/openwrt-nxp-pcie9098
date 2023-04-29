# Building from source

Check the OpenWrt documentation to make sure that you have all the required dependencies available:<br>
<https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem>

<br>

## Set up the source directory
### Clone the OpenWrt repo
#### New setup:
```sh
git clone https://git.openwrt.org/openwrt/openwrt.git
```
Enter the cloned directory:
```sh
cd openwrt
```
Switch to the tag for the version you want to build (`v22.03.4` will be used in this example):
```sh
git checkout v22.03.4
```

#### Existing source directory:
If you have an existing clone, enter the directory and fetch the latest upstream changes:
```sh
git fetch
```
Then reset to the version you're building:
```sh
git reset --hard v22.03.4
```

### Pick the driver package template
```sh
git fetch https://github.com/Ahrotahn/openwrt
git cherry-pick FETCH_HEAD
```

<br>

## Update the feeds
```sh
./scripts/feeds update -a
./scripts/feeds install -a
```

<br>

## Get the kernel configuration
Change `22.03.4` in the URL below to the version you are building:
```sh
wget https://downloads.openwrt.org/releases/22.03.4/targets/mvebu/cortexa72/config.buildinfo -O .config
```
For stable release builds it's *very* important to use the same configuration that was used upstream.
If you are building for a device other than the Mochabin then you would need to use the URL specific for your device.
That can be obtained through <https://downloads.openwrt.org/releases/>

<br>

## Set up the kernel configuration
```sh
make menuconfig
```
In the TUI, double-check that the package is available and marked as a module:
* Kernel modules
  * Wireless Drivers
    * `<M>`kmod-nxp-pcie9098

Exit out and pick `Yes` to save.<br>
If you are planning on making local changes to the package, follow the next section before exiting.

### Local development
This is only necessary if you wish to make local changes to the package and should be skipped if you just want to build with the existing source as-is.

The OpenWrt documentation has a section with the available options for working with local changes at <https://openwrt.org/docs/guide-developer/packages#working_on_local_application_source>

I prefer the source tree override method (as noted in the docs, this requires your changes to be commited).<br>
While in the kernel configuration TUI, from the main menu:
* Advanced configuration options (for developers)
  * Enable package source tree override

Exit out and pick `Yes` to save.

Edit `package/kernel/nxp-pcie9098/Makefile`.<br>
Change `PKG_SOURCE_URL:=` to point to your source.<br>
Add the latest commit hash for your changes to `PKG_SOURCE_VERSION:=` and set `PKG_MIRROR_HASH:=` to `skip`.

<br>

## Modification for stable release builds
Skip this section if you are building a snapshot image. This is only needed to make kernel packages that work with upstream stable releases.

__⚠️ Footgun Warning ⚠️__

This modification is a necessary evil and has the potential to leave you with an unbootable system if you have made any mistakes.

Kernel modules are built against a specific kernel and will only work with that kernel.
In order to prevent conflicts OpenWrt includes a hash inside kernel packages of the environment used to build the kernel and modules.

This driver isn't being built upstream, so simply including it will change the hash which would prevent the package from being installed on upstream releases.
So we'll have to manually override that hash for the package to be installable.

There are very good reasons for this check. Please never make these changes wantonly.


Go to the kmods directory of the OpenWrt release you are building. For `22.03.4` this would be:<br>
<https://downloads.openwrt.org/releases/22.03.4/targets/mvebu/cortexa72/kmods/>

Copy the hash at the end of the kernel version.
The link above should show `5.10.176-1-29e8f127a168a7190cb0593504c20265`, so you'd want to copy `29e8f127a168a7190cb0593504c20265`

Edit `include/kernel-defaults.mk`.
Inside the `define Kernel/Configure/Default` section, the last line in the section should be:

`grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | $(MKHASH) md5 > $(LINUX_DIR)/.vermagic`

Change this to: `echo 'hash' > $(LINUX_DIR)/.vermagic`, replacing `hash` with the one you copied.

<br>

## Build
```sh
make -j$(nproc) defconfig download clean world
```
The output should now be available in `bin/targets/mvebu/cortexa72/packages/`.<br>
For the `230128` build for `v22.03.4`:<br>
`bin/targets/mvebu/cortexa72/packages/kmod-nxp-pcie9098_5.10.176+230128-1_aarch64_cortex-a72.ipk`

