# Intel UHD Graphics 630 Coffee Lake R on macOS

<table>
       <tr><td align=center><img src=img/i9.png></td></tr>
       <tr><td><b>How to set the integrated graphics card Intel UHD Graphics 630 Coffee Lake R (9th gen. i7 and i9) in headless mode (no cable to monitor) to be used by macOS in computing and video encoding tasks or setting it as the main card</b></td></tr>
</table>

**Note**: based on

- `[GUIDE] General Framebuffer Patching Guide (HDMI Black Screen Problem)` by _CaseySJ_ in tonymacx86 forum
- Framebuffer patch feature of _headkaze_'s Hackintool app
- Desktop Coffee Lake chapter of the OpenCore Dortania's guide.

## Preface

Macs with an integrated graphics card (iGPU) and a dedicated one (dGPU) use the iGPU for video encoding and decoding tasks. When building a Hackintosh with both types of GPU we can find that, if the iGPU is not properly installed, video encoding can fail. When this happens, we must configure the iGPU as _headless mode_ (it is so called when it is enabled but no cable to monitor) so that the dGPU acts as main card but the iGPU is available for encoding / decoding video.

iGPU setting depends on 2 factors:

<table>
       <tr><td>Motherboard because manufacturer can put 1, 2 or 3 HDMI connectors for the iGPU on the back pannel</td></tr>
       <tr><td>Intel processor generation, different Intel generations have different iGPU</td></tr>
</table>

My PC has a Z390 Aorus Elite board with an Intel 9th gen. CPU (Coffee Lake Refresh, it is configured as Coffee Lake) with Intel UHD Graphics 630:

<table>
       <tr><td>PCI path is PciRoot(0x0)/Pci(0x2,0x0)</td></tr>
       <tr><td>Plattorm ID is 3E9B0007</td></tr>
</table>

On this board there is only one HDMI v1.4 connector for the iGPU, it corresponds to index 3 in the theoretical list of 3 internal connectors this iGPU can have:

<table>
       <tr><td>Index 1, BusID 0x00, Type HDMI (type does not matter on this port)</td></tr>
       <tr><td>Index 2, BusID 0x00, Type HDMI (type does not matter on this port)</td></tr>
       <tr><td>Index 3, BusID 0x04, Type HDMI (this is the active port, the only physical one, type is HDMI)</td></tr>
</table>

This is important when using the iGPU as a main or single card but not when using it in headless mode.

## Headless mode

- iGPU and dGPU must be enabled in BIOS with dGPU as primary
- There should be no cable between iGPU HDMI port and any type of display
- Lilu and WhatEverGreen properly installed
- SMBIOS iMac19.1.

You have to add in `DeviceProperties >> Add`:

``` xml
<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
<dict>
	<key>AAPL,ig-platform-id</key>
	<data>AwCRPg==</data>
	<key>device-id</key>
	<data>mz4AAA==</data>
	<key>enable-metal</key>
	<data>AQAAAA==</data>
</dict>
```

This code has data values in Base64, in plist editors they can be seen as hexadecimal, e.g. `AwCRPg==` in Base64 (_AAPL,ig-platform-id_) = `0300913E` in hexadecimal.

With these changes you can boot from a dGPU with the iGPU well installed. To check if the VDA Decoder function is activated you can get Hackintool app (_Fully Supported_ or _Failed_ in the first System tab).

Notes:

- `AAPL,ig-platform-id=0300913E` is mandatory, to detect the iGPU as Coffee Lake
- `device-id=9B3E000` to be displayed as `Intel UHD Graphics 630` instead of `Kabylake Unknown` (optional)
- `enable-metal=01` to enable Metal 3 in Ventura (recommendable on macOS 13).

<details>
<summary>Image: iGPU as secondary card</summary>
<br>
<img src="img/iGPU as secondary card.png">
</details>

## iGPU as main card

This card can also be configured to be the main or single one, so that it outputs a signal to the monitor and also encodes video. Here's what to do.

- Enable it on the motherboard as main card: Initial Display Output `IGFX` instead of `PCIe 1 Slot` (this would be the final step)
- Lilu and WhatEverGreen properly installed
- SMBIOS iMac19.1
- Add in `config.plist >> DeviceProperties >> Add` the code below (note: `BwCbPg==` is `07009B3E` in hexadecimal):

``` xml
<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
<dict>
	<key>AAPL,ig-platform-id</key>
	<data>BwCbPg==</data>
	<key>device-id</key>
	<data>mz4AAA==</data>
	<key>device_type</key>
	<string>VGA compatible controller</string>
	<key>enable-hdmi20</key>
	<data>AQAAAA==</data>
	<key>enable-metal</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con0-busid</key>
	<data>AAAAAA==</data>
	<key>framebuffer-con0-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con0-pipe</key>
	<data>EgAAAA==</data>
	<key>framebuffer-con1-busid</key>
	<data>AAAAAA==</data>
	<key>framebuffer-con1-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con1-pipe</key>
	<data>EgAAAA==</data>
	<key>framebuffer-con2-busid</key>
	<data>BAAAAA==</data>
	<key>framebuffer-con2-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con2-pipe</key>
	<data>EgAAAA==</data>
	<key>framebuffer-con2-type</key>
	<data>AAgAAA==</data>
	<key>framebuffer-patch-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-stolenmem</key>
	<data>AAAwAQ==</data>
	<key>hda-gfx</key>
	<string>onboard-1</string>
	<key>force-online</key>
	<data>AQAAAA==</data>
</dict>
```

Note:

- `force-online=01` is mandatory, to force online status on all displays.

In this way, Intel UHD 630 is well installed and works fine on macOS.

<details>
<summary>Image: iGPU as main card</summary>
<br>
<img src="img/iGPU as main card.png">
</details>

**Note about igfxfw and rps-control** (thanks [5T33Z0](https://github.com/5T33Z0))

Many Coffee Lake iGPUs work fine without loading Apple GUC firmware or enabling RPS Control. But those who have graphical issues (low max frequency, fixed frequency that does not vary, screen glitches, etc.) can play with 2 available properties:

- Apple GUC (disabled by default): it can be enabled by `igfxfw` property (02 as Data) or `igfxfw=2` boot arg. It forces loading of Apple Graphics Unit Control (GUC) firmware (graphics accelerator uses Apple firmware scheduler). It improves Intel Quick Sync Video and iGPU performance. Ir requires chipset supporting Intel Management Engine v12 or newer (H310, C246, B360, H370, Q370, Z390, Z490, etc.). It's buggy and not advisable to use. Should be tested on a case by case basis.
- RPS Control (disabled by default): it can be enabled by `rps-control` property (01 as Data) or `igfxrpsc=1` boot arg. It enables RPS control patch (graphics accelerator uses host preemptive scheduler). Improves iGPU performance on Kaby Lake and newer using older chipsets without Intel Management Engine v12 support (Z370 and others). `rps-control`property was enabled in WhateverGreen until macOS 10.15.6, in this version it had major issues and, since then, WhateverGreen has disabled it by default.

Recommendations:

- If the iGPU has normal frequencies and power consumption for that model and the screen works fine >> it is not necessary to use any of the 2 settings
- Both should not be used at the same time because they are based on different modes of operation
- `igfxfw` takes precedence over `igfxrpsc` when both are set
- Boards supporting Intel ME 12 and newer can try `igfxfw=1`
- For boards with older chipsets, RPS-Control is an option
- There are comments of OC developers saying that `rps-control` is better than `igfxfw` (Big Sur and newer) because of the bug that seems to be in Apple GUC.

**Note about KP waking from sleep**

If you have KP or black screen when macOS wakes from sleep, you have to replace `hda-gfx` property with `No-hda-gfx`, this usually fixes those KPs but audio through HDMI is lost. Replace:

``` xml
<key>hda-gfx</key>
<string>onboard-1</string>
```

with:

``` xml
<key>No-hda-gfx</key>
<data>AAAAAAAAAAA=</data>
```

**Note about QEMU**

If macOS is being installed in a QEMU virtual machine, the primary card in BIOS must be set to dGPU. Otherwise `vesafb` grabs the iGPU which, due to a memory allocation conflict, results in a long delay on startup/shutdown and multi-gigabyte logs.

If `vga=off` is added to grub, then the card is not claimed, however it also does not work (blank screen).
