# CMP 70HX mining unlock for HiveOS

[Русская версия](README.ru.md)

> **Purpose.** This repository documents the known working CMP 70HX mining-unlock setup for HiveOS: required versions, prerequisites, installation order, validation, and operating checks. It is **not** a VBIOS-flashing guide and does not add physical video outputs.

## Known working versions

The recorded setup used the following exact combination:

| Component | Required version / value |
| --- | --- |
| HiveOS kernel | `6.6.0-hiveos` |
| NVIDIA driver | `610.43.03` |
| NVIDIA module type | Open kernel modules |
| Unlock patch | `0014-cmp70hx-ga104-rejoin14.patch` |
| Successful-load marker | `PASS_CMP70HX_RUNNING` |
| Miner used after validation | WildRig |

Do **not** substitute a newer driver, another NVIDIA branch, a proprietary-module build, or a patch made for a different kernel. The versions above work as one set.

## What you need before installation

- A CMP 70HX rig that you own or are authorised to administer.
- HiveOS with the exact kernel `6.6.0-hiveos` already running.
- NVIDIA Open Kernel Modules source for exactly `610.43.03`.
- The verified patch file `0014-cmp70hx-ga104-rejoin14.patch` made for that exact source tree.
- The matching kernel headers and build tools on the target rig.
- A way to access the rig after reboot (HiveOS dashboard, local console, or SSH).
- A backup of the current working configuration before changing the driver.

The patch and NVIDIA source are **not** included here. Do not use a patch downloaded from an unknown site. To make this project fully one-command reproducible, add the verified source reference, patch checksum, and its original build script to this repository under `patches/` and `scripts/`.

## Pre-flight check

Run these commands on the rig and keep the output with your install notes:

```bash
uname -r
lspci -Dnn | grep -i nvidia
nvidia-smi -L
modinfo -F version nvidia
```

Continue only when the running kernel and driver source/patch are the exact versions listed above. Stop if the card is not detected or if a different driver is loaded.

## Installation procedure

1. Disable automatic changes for this rig while installing. In particular, do not run `nvidia-driver-update` and do not upgrade HiveOS.
2. Place the verified NVIDIA `610.43.03` Open Kernel Modules source and `0014-cmp70hx-ga104-rejoin14.patch` on the rig.
3. Apply the patch only to the matching, clean source tree.
4. Build the module on the **same rig** and against the currently running `6.6.0-hiveos` kernel. Do not copy a compiled `nvidia.ko` from another machine.
5. Install the build using the original build procedure supplied with the verified source bundle.
6. Reboot the rig after the install completes without errors.

The original patch bundle and its build script are required for the exact build commands. This guide deliberately does not invent commands or publish an unverified patch: an incorrect NVIDIA module can leave the rig without a working driver.

## Verify the unlock before mining

After reboot, run:

```bash
dmesg -T | grep -E 'PASS_CMP70HX_RUNNING|NVRM|nvidia' | tail -100
nvidia-smi -L
nvidia-smi --query-gpu=name,driver_version,pstate,power.draw,temperature.gpu,utilization.gpu --format=csv,noheader
```

The expected result is:

- `PASS_CMP70HX_RUNNING` appears in the kernel log;
- `nvidia-smi` detects `NVIDIA CMP 70HX`;
- the driver reports version `610.43.03` and the GPU can enter `P0` under load.

Only then start WildRig or another miner you have configured, and confirm stable hashrate, temperatures, power draw, and accepted shares.

## Known operating profile

One recorded CMP 70HX profile produced about **43.04 TH/s** on PearlHash:

| Setting | Recorded value |
| --- | --- |
| HiveOS power limit | 205 W |
| Actual draw | about 178 W |
| Core clock | about 1455 MHz |
| GPU temperature | about 68 °C |
| Fan speed | about 83% |

This is a reference point for one card, miner, and room temperature—not a universal overclock. Change one setting at a time and monitor temperature, power, rejected shares, and driver messages.

## Unlock troubleshooting

| Symptom | What to check |
| --- | --- |
| `PASS_CMP70HX_RUNNING` is absent | Confirm the exact kernel, Open Module version, patch, and clean build log. |
| `nvidia-smi` cannot see the card | Check PCI detection, loaded NVIDIA module, and recent `NVRM` lines in `dmesg`. |
| The driver fails after an update | Revert to the recorded `6.6.0-hiveos` / `610.43.03` combination; do not mix components. |
| Hashrate is unstable | Return to conservative clocks, check power/temperature, and inspect the miner log. |

## Safety notes

- Do not flash the VBIOS as part of this procedure.
- Do not publish secret pool credentials, wallet addresses you want kept private, or remote-access keys.
- Do not share non-public NVIDIA source or patches unless you are allowed to distribute them.
- Test one controlled reboot after the rig is stable, so you know the module loads correctly at startup.
