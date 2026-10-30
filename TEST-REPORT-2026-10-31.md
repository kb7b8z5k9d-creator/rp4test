# RP4 Pro DroidDeck — 2026-10-31 update: PanVK, Vulkan 1.3 and D3D12

Device: Retroid Pocket 4 Pro — Mali-G77 MC9, Android 13, 1334×750 @ 60 Hz, no root.
Build: `yujikaido/droiddeck-rp4pro` fix51 + local changes, self-signed (`com.rp4pro.droiddeck.lab`).
Latest build: APK SHA-256 `c1edd69d24ff4e531516a9e7e944ac0e3c3cdd8e8e760bbc7f0ed8a94abc677a`
(pre-release `v0.3.0-dx12-preview`), plus a VKD3D component
`vkd3d-3.1-readback-fallback.wcp` (SHA-256 `3c3043bdb57f18a7bc8b93f36acf846e35693f7eb4c7b97f2fb7f5922dc8cfa6`).

Source: https://github.com/kb7b8z5k9d-creator/rp4develop
Release: https://github.com/kb7b8z5k9d-creator/rp4test/releases/tag/v0.3.0-dx12-preview

---

## 1. PanVK on Mali-G77 (faux123 v1.1.3)

- The imported driver reports **Vulkan 1.3.363** on the Mali-G77 (stock is 1.1.177), with
  `VK_EXT_transform_feedback` (`transformFeedback=1`, **`geometryStreams=1`**),
  `VK_ANDROID_external_memory_android_hardware_buffer`, dma-buf export/import, AHB, and 178 device
  extensions.
- **Android's Vulkan loader ignores `VK_ICD_FILENAMES`.** Loading the ICD through the environment
  variable leaves the system Mali driver in place. The vkbridge server now loads the selected ICD
  itself: `--vulkan <lib>` plus a fallback from `vkGetInstanceProcAddr` to
  `vk_icdGetInstanceProcAddr` (the entry point an Android ICD exports).
- The driver manager's existing import path accepts faux123's `.adpkg.zip`
  (`libvulkan_panfrost.so` + `meta.json`) without changes to the UI.

## 2. dma-buf export probe and automatic AHB fallback

- On PanVK, the LINEAR modifier image query claims exportability (features `0x6`), but the real
  `vkGetMemoryFdKHR` returns `-2`. A direct query alone is not enough.
- The server probe (`server_probe.c`, PROBE_VERSION 3 → 4) now creates a real image when OPTIMAL
  does not report exportability, tries a LINEAR modifier image and an actual export, and on failure
  enables the existing AHB-page export emulation automatically. No forced debug switch is needed.
- Result: Steam/gamescope start on PanVK; the earlier `dma-buf image query refused` /
  `vkGetMemoryFdKHR failed` crash at gamescope start is gone.

## 3. VKD3D: READBACK with no HOST_CACHED type

- Little Nightmares Enhanced Edition (D3D12) failed with
  `Found no suitable memory type for requested type_flags #2`. With the exact VKD3D-Proton version
  used by Proton, the failing allocation was a READBACK heap type 3, 16 MiB, with base mask 0.
- The driver exposes AHB-shared coherent type 1 (`0x7`), but **no usable HOST_CACHED type**
  (type 2 `0xb` fails AHB import; the cached-readback self-test is `0x0`, so the feature cannot be
  claimed).
- VKD3D's suballocation path pre-filters HOST_CACHED and ends with an empty candidate set, so its
  own optional-cache fallback never runs. The general fix adds
  `VK_MEMORY_PROPERTY_HOST_CACHED_BIT` to the optional flags of the suballocation: it still prefers
  cached memory, and falls back to the real HOST_VISIBLE type when none exists. It does not
  advertise a feature the driver lacks.
- Patch and reproduction: `src/tools/vkd3d/patches/0001-readback-host-cached-fallback.patch`,
  packaged as a component by `src/tools/vkd3d/package-component.sh` (same package rebuilt → same
  SHA-256).

## 4. DXVK selection for an imported ICD

- `AppVulkan.detect` compared the imported PanVK report (Vulkan 1.3) with the system Mali driver
  (1.1) and discarded it, selecting an old DXVK (1.10.3) instead — Little Nightmares then failed
  with `DXGI_ERROR_UNSUPPORTED` (`CreateSwapChainForHwnd: Unsupported device type`).
- Fix: an imported ICD is cached by the library's SHA-256, its report is read without being
  compared to the system driver, it gets its own server/socket, and the DXVK component is
  re-selected after the bridge self-test. The session now reads
  `Vulkan 1.3, shaderInt64 yes, push constants 256 (bridge)`, withdraws DXVK 1.10.3 and restores
  Proton's own DXVK.

## 5. Results on the device

| Test | Result |
|---|---|
| Steam / gamescope on PanVK | starts (automatic AHB fallback) |
| Little Nightmares Enhanced Edition (D3D12, VKD3D) | enters the game, renders correctly, no flicker; **low frame rate** (one Win-FG 2× reading 11 → 22 fps) |
| ForestTD (Steam 4969280) **default D3D12** | runs and plays normally; ~45.6 fps with Win-FG 2× (about 23 native) |
| ForestTD via Vulkan (`--rendering-method mobile`) | runs at ~50 fps in the tested scene; the temporary launch option was removed afterwards |

## 6. Scope and open items

- Validated: 64-bit path, one Proton (`Proton Experimental (ARM64)`), two D3D12 games plus the
  Vulkan path above.
- **Not validated:** 32-bit VKD3D, other Proton versions, other games, switching back to the
  system Mali driver, and performance.
- Performance is not optimised: the low frame rate coincides with low available memory and heavy
  shader compilation; the bottleneck is not proven.
- No game-specific DLLs or environment variables are left in place. The VKD3D fix is a general
  component; PanVK itself remains an experimental driver that must be imported separately.
