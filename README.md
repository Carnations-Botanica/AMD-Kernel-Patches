<span align="center">

# AMD Kernel Patches for OpenCore
###### Try these patches at your own risk, and always keep an EFI backup.

</span>

# Purpose

Binary kernel patches to enable almost native AMD CPU support on OS X.

## Support Chart

| Release Name | Status | Notes |
| --- | --- | --- |
| Sierra | <span style="color: #7afc4e;">Complete</span> | None |
| El Capitan | <span style="color: #7afc4e;">Complete</span>  | None |
| Yosemite | <span style="color: #7afc4e;">Complete</span> | None |
| Mavericks | <span style="color: #7afc4e;">Complete</span> | Requires a TSC Syncing Kext or you will kernel panic. |
| Mountain Lion | <span style="color: #7afc4e;">Complete</span> | Requires rebuilding kernelcache for DEBUG variant builds. |
| Lion | <span style="color: #7afc4e;">Complete</span> | TSC Sync Issues, boots but unstable and is practically unusable. |
| Snow Leopard | <span style="color: #ffe985;">Work-In-Progress</span> | KernelSpace is reached, but TSC issues do not allow further boot. |
| Leopard | <span style="color: #a80000;">Incomplete</span> | None |
| Tiger | <span style="color: #a80000;">Incomplete</span> | None |

## Gallery

<h3 align="center">macOS Sierra 10.12.6 (16G29)</h3>
<p align="center">
  <img src="./assets/gallery/Sierra.png">
</p>

<h3 align="center">OS X El Capitan 10.11.6 (15G31)</h3>
<p align="center">
  <img src="./assets/gallery/El-Capitan.png">
</p>

<h3 align="center">OS X Yosemite 10.10.5 (14F27)</h3>
<p align="center">
  <img src="./assets/gallery/Yosemite.png">
</p>

<h3 align="center">OS X Mavericks 10.9.5 (13F34)</h3>
<p align="center">
  <img src="./assets/gallery/Mavericks.png">
</p>

<h3 align="center">OS X Mountain Lion 10.8.5 (12F45)</h3>
<p align="center">
  <img src="./assets/gallery/Mountain-Lion.png">
</p>

<h3 align="center">OS X Lion 10.7.5 (11G63)</h3>
<p align="center">
  <img src="./assets/gallery/Lion.png">
</p>

<h3 align="center">OS X Snow Leopard 10.6.7 (10J3250)</h3>
<p align="center">
  <img src="./assets/gallery/Snow-Leopard.png">
</p>

# Preliminary Information

### Custom OpenCore Requirement

If you want to test these versions of OS X / macOS on your own machine, as of right now you'll be required to use a modified copy of [OpenCore](https://github.com/Carnations-Botanica/OpenCorePkg/actions/runs/15984994642) by Carnations Botanica that has changes to MIN/MAX of default patches that affect AMD CPUs and VMs. These caused the initial early kernel panics in earlier testings.

**You must be signed in to download Github Artifacts like the required <code>macOS XCODE5 Artifacts</code>!**

### OpenCore Quirks

Ensure ``FixupAppleEfiImages`` quirk is enabled to ensure W^R errors on older OS X don't appear.

Ensure the Kernel Quirk `ProvideCurrentCpuInfo` is enabled. OpenCore 0.7.1 or newer is required. You should NOT be using an outdated copy of OpenCore, this requirement has long been deprecated. Make sure to **enable** this quirk or the system **won't boot**. You're only warned once.

### Patch List

Depending on the specific property list you use for your target OS X installation, you can get any of the following patches that are backported from High Sierra, or new to the scene with Legacy OS X:

| Function/Target | Patch Name | Comment |
| --- | --- | --- |
| _cpuid_set_generic_info | Remove wrmsr(0x8B) | None |
| _cpuid_set_generic_info | Replace rdmsr(0x8B) with constant 186 | None |
| _cpuid_set_generic_info | Set flag=1 | None |
| _cpuid_set_generic_info | Disable check for Leaf 7 | None |
| _cpuid_set_cpufamily | Force CPUFAMILY_INTEL_PENRYN | None |
| _cpuid_set_cache_info | CPUID 0x8000001d instead of 4 | AMD uses a different Leaf than Intel. |
| _cpuid_set_info | cpuid_cores_per_package set to const | Manually set because detection fails on AMD |
| _commpage_populate | Remove rdmsr | None |
| _i386_init/_commpage_populate/_pstate_trace | Remove rdmsr calls | Various places. |
| _lapic_init | Remove version check panic | None |
| _mtrr_update_action | Set PAT MSR to 00070106h | This patch is only for 10.10+ |
| _panic_epilogue | Prevent instant reboot on panic | Allows for the CPU halt to be avoided |
| mp.c | Increase TSC sync delta margin to prevent panic call | On 10.9- TSC Sync issues are so dangerous that they can cause instant panic and reboots. |
| String Replace | GenuineIntel to AuthenticAMD | None |

### Configuring cpuid_cores_per_package patch.

The Core Count per Package patch needs to be modified to boot your system. The first kernel patch is required to be updated manually according to these instructions no matter what property list you choose. Update the `Replace` value only.

| OS X Version | Default Value | Example Value |
| --- | --- | --- |
| 10.12 | BA 00 00 00 00 00 | BA 02 00 00 00 00 |
| 10.11 | BA 00 00 00 00 00 | BA 04 00 00 00 00 |
| 10.10 | 41 BE 00 00 00 00 | 41 BE 06 00 00 00 |
| 10.9 | BA 00 00 00 00 | BA 08 00 00 00 |
| 10.8 | B8 00 00 00 00 | B8 0C 00 00 00 |
| 10.7 | TBD | TBD |
| 10.6 | TBD | TBD |
| 10.5 | TBD | TBD |
| 10.4 | TBD | TBD |

From the table above, replace `<BX XX>` with the hexadecimal value matching your physical core count. Do not use your CPU's thread count. See the table below for the values matching your CPU core count.

| Core Count | Hexadecimal |
| --- | --- |
| 2 Cores | `02` |
| 4 Cores | `04` |
| 6 Cores | `06` |
| 8 Cores | `08` |
| 12 Cores | `0C` |
| 16 Cores | `10` |
| 24 Cores | `18` |
| 32 Cores | `20` |

## Features

- Leverages OpenCore to run OS X on AMD CPUs without a custom built kernel.

## Disadvantages

- No 32-bit support (OPEMU)

## Notes for Users

- Currently, Mavericks and below require the usage of a stock but DEBUG variant Kernel. These can be found under ``extras/kernels/*``. To use these, simply apply them to your installation media and rebuild the kernelcache. You must/can/may end up needing to equally add the DEBUG Kernel as a post-install step, before you can boot into the final installation. This will be resolved once we can get more testers with real serial output to get us the kernel panics that we need, to create the missing RELEASE patches.

## Supported AMD CPUs

As of right now, these are all theoretically supported AMD CPUs. Testing is greatly appreciated, and opening an Issue on the repo with DEBUG logs is equally appreciated. It seems that some generations of AMD CPUs have TSC issues bad enough to not allow the machine to boot. Please test extensively and send in reports!

| Family | Codename | Product Name |
| --- | --- | --- |
| 17h and 19h | Zen | Ryzen, Threadripper, Athlon 2xxGE |
| 16h | Jaguar | A Series (including AM4 A-Series) |
| 15h | Bulldozer | FX Series |

## AMD Kernel Patches Credits

If any credits are missing, they are to be added in future commits as the project wraps up.

- [RoyalGraphX](https://github.com/RoyalGraphX) for the idea to add support for older OS X releases, main backporting effort such as updating PAT for Sierra, researching cpuid_cores_per_package on older OS X releases, and testing on baremetal machines.

- [Zormeister](https://github.com/zormeister) for the idea to add support for older OS X releases, initial patch matching confirmations for High Sierra -> Sierra backporting effort, cpuid_cores_per_package effort

- [Dhinak G](https://github.com/dhinakg), helping reverse-engineer functions for new Find/Replace values in Tiger, for CPUID 4 and Cores Per Package patches

- [Shaneee](https://github.com/shaneee), helping tackle Snow Leopard Kernel Patches, and assisting in building XNU releases for binary diffing, updating Force PENRYN patches with Masks for wider support, compiling the Mavericks 10.9.5 ``mach_kernel`` as ``DEBUG`` variant for improved serial output logging.

- [Shantonu](https://shantonu.blogspot.com/) of ssen's blog, which was a huge help, and always has been when it comes to knowing exactly what is required to build the XNU kernel for each of XNU's major releases since Snow Leopard. This was major when building vanilla, but DEBUG/DEVELOPMENT Kernels to test and get better serial output from. 

- []()

## AMD Vanilla Credits

- [AlGrey](https://github.com/AlGreyy) for the idea and creating the patches.

- [XLNC](https://github.com/XLNCs) for maintaining patches for various macOS versions.

- [Acidanthera](https://github.com/acidanthera) for OpenCore.

- [CaseySJ](https://github.com/CaseySJ/) for Zen 4 IOPCIFamily patches.

- Sinetek, Andy Vandijck, spakk, Bronya, Tora Chi Yo, [Shaneee](https://github.com/Shaneee) and many others for sharing their AMD/XNU kernel knowledge.

<h6 align="center">A big thanks to all contributors and future contributors! ꩓</h6>
