# Patch-Recovery

Samsung recovery image patching with GitHub Actions, intended to enable **fastbootd** on compatible devices.

![Android](https://img.shields.io/badge/Android-3DDC84?logo=android&logoColor=white)
![Shell](https://img.shields.io/badge/Shell-121011?logo=gnubash&logoColor=white)
[![RECOVERY workflow](https://github.com/MrDoctorAS/Patch-Recovery/actions/workflows/recovery.yml/badge.svg?branch=master)](https://github.com/MrDoctorAS/Patch-Recovery/actions/workflows/recovery.yml)
[![VENDOR_BOOT workflow](https://github.com/MrDoctorAS/Patch-Recovery/actions/workflows/vendor_boot.yml/badge.svg?branch=master)](https://github.com/MrDoctorAS/Patch-Recovery/actions/workflows/vendor_boot.yml)

[Workflows](https://github.com/MrDoctorAS/Patch-Recovery/actions) · [Releases](https://github.com/MrDoctorAS/Patch-Recovery/releases) · [Report an issue](https://github.com/MrDoctorAS/Patch-Recovery/issues)

## Overview

This repository automates downloading, unpacking, patching, and repacking Samsung images. It builds on [phhusson's Samsung Galaxy A51 GSI boot work](https://github.com/phhusson/samsung-galaxy-a51-gsi-boot). Compatibility depends on the device, firmware, and recovery layout; fastbootd support is not guaranteed.

| Workflow | Required input | Intended release file |
| --- | --- | --- |
| `RECOVERY` | `RECOVERY_URL`: direct URL to `recovery.img` or its LZ4-compressed version | `fastbootd-recovery.tar.md5` |
| `VENDOR_BOOT` | `VENDOR_BOOT_URL`: direct URL to `vendor_boot.img` or its LZ4-compressed version | `fastbootd-vendor_boot.tar.md5` |

These workflows publish files to **GitHub Releases**, not an Actions ZIP artifact. A workflow badge reports GitHub's recorded run status, not device compatibility or image safety.

## Safety and prerequisites

> **Warning:** Flashing modified firmware can erase data, prevent booting, or permanently affect device security features. Disabling verified boot reduces security. Only modify devices you own or are authorized to service, and do not flash an output merely because a workflow finished successfully.

| Requirement | Details |
| --- | --- |
| Matching firmware | Use an image for your exact device model and firmware build. Do not substitute another device's image. |
| Recovery plan | Back up important data and obtain matching stock firmware and device-specific restoration instructions before proceeding. |
| Bootloader compatibility | Verify that your device permits the intended modification. Bootloader unlocking can erase data and may affect Knox or warranty coverage. |
| GitHub Actions | Use your own fork with Actions enabled and enough Actions quota. The workflows run on `ubuntu-latest`. |
| Download URL | Use a trusted, direct download accessible without interactive login. A sharing page is not necessarily a file URL. |
| URL privacy | Do not provide credentials, private tokens, or sensitive signed URLs: inputs can appear in logs, and files may be published in releases. |
| Release permissions | Release publication requires appropriate repository token permissions, typically `contents: write`; organization policy may restrict this. |

## Usage

| Step | Action |
| --- | --- |
| 1 | Fork this repository and review the workflow and scripts before enabling Actions. |
| 2 | Extract the matching recovery or vendor boot image from your firmware and host it at a trusted direct-download URL. |
| 3 | Open **Actions** in your fork and choose **RECOVERY** or **VENDOR_BOOT**, according to your device's partition layout. |
| 4 | Select **Run workflow**, choose the intended branch, and replace the example URL with your actual image URL. |
| 5 | Review every job step and check for unpacking, patching, signing, or packaging errors. See the recovery limitation below. |
| 6 | If release publication succeeds, open **Releases** in your fork. The workflows use the run ID as the release tag and publish a `.tar.md5` file. |
| 7 | Inspect the archive and verify its device suitability before following a trusted, device-specific flashing guide. |

The workflows name their releases `Patched-Recovery` and `Patched-Vendor_Boot`. The generated archives contain `recovery.img` and `vendor_boot.img`, respectively.

### Important recovery workflow limitation

The current `RECOVERY` workflow passes `recovery-patched.img` to `avbtool add_hash_footer`, but `script2.sh` creates `r-patched.img`. That filename mismatch can stop the job before packaging or release publication. This README update does **not** fix the workflow or claim that the recovery build is working.

Both workflows also invoke the patch scripts with `|| true`, so some earlier errors may be suppressed. Inspect logs and outputs rather than relying only on a green badge.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| Downloaded image is invalid | Ensure the URL returns the image rather than HTML, a login page, or a hosting-site error. |
| No **Run workflow** button | Ensure Actions is enabled in your fork and the selected branch contains the workflow. |
| Unpacking or patching fails | Verify the input format and device layout. The included patches are not universal. |
| Missing `recovery-patched.img` | Review the known `RECOVERY` filename mismatch above; do not treat a missing output as a successful patch. |
| No release appears | Inspect the job result and release-upload step; confirm the repository token can publish releases. |
| Unsure about vbmeta or Odin slots | Follow a guide for your exact device. The included `vbmeta_disabled_R.tar` is not a universal flashing requirement. |

## Repository guide

| Path | Purpose |
| --- | --- |
| `.github/workflows/recovery.yml` | Manually triggered recovery-image workflow. |
| `.github/workflows/vendor_boot.yml` | Manually triggered vendor boot workflow. |
| `script1.sh` | Prepares `r.img` from raw or LZ4 input and generates a local key if needed. |
| `script2.sh` | Unpacks the image, applies recovery binary patches, and writes `r-patched.img`. |
| `magiskboot` | Bundled tool used for image and ramdisk processing. |
| `avbtool` | Bundled Android Verified Boot utility used by the recovery workflow. |
| `vbmeta_disabled_R.tar` | Existing auxiliary archive; confirm applicability before considering use. |

## Reporting problems

Open an issue with the device model, firmware build, selected workflow, input format, and a link to the failing run or a sanitized log excerpt. Do not publish device identifiers, secrets, or private download links. If proposing a script fix, include reproducible evidence and explain the expected behavior change.

## Credits and related work

| Project or contributor | Acknowledgment |
| --- | --- |
| [phhusson](https://github.com/phhusson) | Foundational [Samsung Galaxy A51 GSI boot scripts](https://github.com/phhusson/samsung-galaxy-a51-gsi-boot). |
| [James Nguyen / thongass000](https://github.com/thongass000) | Script simplification and improvements credited in the original README. |
| [engineer4t/fastboot-patcher](https://github.com/engineer4t/fastboot-patcher) | Related patching script referenced by the original README. |

Upstream authors retain credit for their work. Refer to the relevant upstream projects for their terms; this documentation does not grant a new license for bundled tools or firmware.
