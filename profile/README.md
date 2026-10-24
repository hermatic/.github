<div align="center">

<img src="https://raw.githubusercontent.com/hermatic/.github/main/profile/hermatic-logo.png" width="128" alt="Hermatic logo">


# HERMATIC

**The immutable operating substrate.**

*Systems without drift.*

![Status: pre-release](https://img.shields.io/badge/status-pre--release-333333?style=flat-square&labelColor=000000)
![Validated in QEMU](https://img.shields.io/badge/validated-QEMU-333333?style=flat-square&labelColor=000000)
![License: MIT](https://img.shields.io/badge/license-MIT-333333?style=flat-square&labelColor=000000)

</div>

Hermatic is an **image-based Linux OS** built from Arch packages using **mkosi** and **systemd**. The host is sealed: verified, read-only `/usr`; signed boot; encrypted root; and atomic updates. Development runs alongside the host in disposable, user-controlled containers.

> **Pre-release:** Validated in QEMU, not yet on bare metal. No public release and **not ready for production use**.

## Guarantees

| Guarantee | How it works |
| :-- | :-- |
| **Boot what you built** | A signed unified kernel image (UKI) covers the kernel, initrd, and command line. Secure Boot checks it, while dm-verity verifies the read-only `/usr` on every read. Two clean builds produce a bit-identical `/usr`. |
| **Update without partial state** | `systemd-sysupdate` writes the inactive A/B slot. Boot counting automatically falls back to the previous slot if the new one cannot boot. |
| **Unlock only on trusted boot** | LUKS2 root unlocks via TPM2 under a signed PCR 11 policy with PCR 15 = 0. Other boot states require the recovery key. |
| **Keep the host clean** | `hermatic devbox` launches `systemd-nspawn` development containers with `$HOME` bind-mounted. Install tools in a container without changing the host; desktop apps use Flatpak. |

## At the console

```console
$ hermatic verify
Hermatic system integrity
──────────────────────────────────────────────────
  ✓ verity hash     ok
  ✓ UKI signature   ok
  ✓ slot integrity  ok
  ✓ pcrlock state   ok
──────────────────────────────────────────────────
  ✓ overall         ok
```

The `hermatic` CLI is written in Rust and provides:

`status` · `update` · `rollback` · `verify` · `install` / `installer` · `devbox` · `apply`

### Boot chain

```text
firmware → systemd-boot → signed UKI → dm-verity /usr → LUKS2 root (TPM2) → userspace
```

## Project status

| Stage | Status | Scope |
| :-- | :-- | :-- |
| QEMU validation | **Passed** | Installer → TPM2 unlock → two A/B updates → rollback, using a persistent TPM. 100/100 install soak-test cycles passed. |
| Bare-metal validation | **Next** | Validation on physical hardware. |
| `v1.0.0` · `hermatic-server` | **Planned** | Core OS and `systemd-nspawn` tooling. |
| `hermatic-desktop` | **Later** | Hyprland and Flatpak, on a separate branch (v1.1 or v2.0). |

**Current hardware scope:** AMD CPU microcode, AMD GPU firmware, and Realtek NIC firmware. Wi-Fi and Intel firmware are not yet included.

## Deliberate boundaries

- **Not a general-purpose distro.** Packages are resolved at build time; there is no runtime package manager on the host.
- **Not a Docker or Podman host.** Containers are `systemd-nspawn` only.
- **Not designed for proprietary NVIDIA drivers or kernel-level anti-cheat.** Secure Boot and verity rule out unsigned kernel modules.
- **No telemetry.** Hermatic does not phone home.

## Technology

`Arch` · `mkosi` · `systemd 262` · `systemd-boot` · `dm-verity` · `LUKS2` · `TPM2` · `systemd-sysupdate` · `systemd-nspawn` · `Flatpak` · `Rust`

---

Inspired by Lennart Poettering's [*Fitting Everything Together*](https://0pointer.net/blog/fitting-everything-together.html).

<sub>OS repository private until v1.0.0 · MIT license</sub>
