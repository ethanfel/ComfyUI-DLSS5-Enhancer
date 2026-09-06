The direct NGX bridge and caller shim are adapted from
[kos94ok/ComfyUI-DLSS5-NR-Linux](https://github.com/kos94ok/ComfyUI-DLSS5-NR-Linux),
based on [lisitskyaa/ComfyUI-DLSS5-NR](https://github.com/lisitskyaa/ComfyUI-DLSS5-NR).
Their MIT license is preserved in `LICENSE`.

`frame_worker.cpp` implements this pack's version-4 pipe protocol. The bridge
runs the DLAA/SR carrier at every scale so the existing model-preset control
also applies at 1x. It calls Neural Rendering directly, without ReShade.

Build on Linux with MinGW-w64: `bash native/build_linux.sh`.
The outputs in `runtime/` are Windows binaries executed by Wine. NVIDIA
libraries and the Wine/VKD3D/DXVK-NVAPI installation remain external dependencies.

For a fresh Linux setup:

1. Install the Python requirements in ComfyUI's environment and provide the
   runtime with `python install_runtime.py --yes` (or `--runtime-dir PATH`).
2. Run `bash native/build_linux.sh`. When using an external runtime directory,
   copy the resulting `linux/` and `caller/` directories into it.
3. Copy the driver's `/usr/lib/nvidia/wine/_nvngx.dll` beside `nvngx_dlssnr.dll`.
4. Initialize a dedicated Wine prefix with
   `WINEPREFIX=/absolute/path/to/runtime/wineprefix wineboot -u`.
5. In that prefix's `drive_c/windows/system32`, install the 64-bit
   `d3d12.dll` and `d3d12core.dll` from VKD3D-Proton, `dxgi.dll` from DXVK,
   and `nvapi64.dll` from DXVK-NVAPI. These can come from an installed Proton
   build's `files/lib/wine` directory. Keep the default `Z:` mapping to `/`.
6. Set `wine_executable` and `wine_prefix` in the node pack's `config.json` if
   the defaults (`wine` on PATH, `runtime/wineprefix`) do not match the setup.
   Set `ffmpeg_dir` to a directory with native Linux FFmpeg/FFprobe if needed.
7. Run `python selftest.py --frames 3`, then restart ComfyUI.

Local validation on 2026-09-06 used an RTX 5090, NVIDIA driver 610.57.04,
Wine 11.17, and the graphics DLLs from Proton CachyOS 11.0 (20260521).
The installed ComfyUI was `/media/p5/Comfyui`, with Conda environment
`13_env_py313`. Image-batch rendering with warmup and 2x HEVC video rendering
both completed through ComfyUI with feature-18 verification enabled.

The bridge also explicitly shuts down the directly initialized NR snippet
before unloading it. NGX core shutdown alone does not release that session.
Other GPU/driver/runtime combinations and pixel parity with RenoDX have not
been validated.
