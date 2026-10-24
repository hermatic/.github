```
███████
███████          ██
███████          ██
███████          ██
███████          ██      H E R M A T I C
███████ ████████ ██
███████          ██      the immutable operating substrate
███████          ██
███████          ██
███████          ██
███████
```

![status: pre-release](https://img.shields.io/badge/status-pre--release-333333?style=flat-square&labelColor=000000)
![validated in QEMU](https://img.shields.io/badge/validated-QEMU-333333?style=flat-square&labelColor=000000)
![not ready for production use](https://img.shields.io/badge/production-not%20ready-333333?style=flat-square&labelColor=000000)
![MIT](https://img.shields.io/badge/license-MIT-333333?style=flat-square&labelColor=000000)

### *Systems without drift.*

Hermatic is an image-based Linux OS built from Arch packages with mkosi and systemd. The host is sealed: a read-only, verified `/usr`, a signed boot, an encrypted root and atomic updates. Development happens next to it, in containers that are yours to break.

Inspired by Lennart Poettering's ["Fitting Everything Together"](https://0pointer.net/blog/fitting-everything-together.html).

---

### What it promises

**The OS you boot is the OS that was built.**
Kernel, initrd and command line are signed as one unified kernel image (UKI), checked by Secure Boot with the project's own keys. `/usr` is read-only and verified by dm-verity on every read. Two clean builds give a bit-identical `/usr`.

**An update never leaves you half-updated.**
systemd-sysupdate writes the new version into the slot that isn't running. If the new slot fails to boot, boot counting falls back to the old one on its own.

**The disk opens only for a signed Hermatic boot.**
Root is LUKS2, unlocked by the TPM2 under a signed PCR 11 policy with PCR 15 = 0. Anything else needs the recovery key.

**Your tools stay yours.**
`hermatic devbox` gives you systemd-nspawn development containers with `$HOME` bind-mounted. Install anything inside; the host doesn't change. Desktop apps come from Flatpak.

---

### At the console

```
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

The `hermatic` CLI is written in Rust: `status` · `update` · `rollback` · `verify` · `install` / `installer` · `devbox` · `apply`.

```
firmware → systemd-boot → UKI → dm-verity /usr → LUKS2 root (TPM2) → userspace
```

---

### Where it stands

```
✓  QEMU      installer → TPM2 unlock → two A/B updates → rollback,
             with a persistent TPM; 100-cycle install soak test, 100/100
→  next      bare-metal validation
·  v1.0.0    hermatic-server     core + nspawn tooling
·  after     hermatic-desktop    Hyprland + Flatpak, on a branch (v1.1 or v2.0)
```

No release yet, and **not ready for production use**. Hardware today: AMD CPU microcode, AMD GPU firmware and Realtek NIC firmware; no Wi-Fi and no Intel firmware yet.

---

### What it is not

- **A general-purpose distro.** There is no runtime package manager; packages are a build-time tool.
- **A Docker or podman host.** Containers are systemd-nspawn only.
- **For NVIDIA proprietary drivers or kernel-level anti-cheat.** Secure Boot and verity rule out unsigned kernel modules.
- **Something that phones home.** No telemetry.

---

`Arch` · `mkosi` · `systemd 262` · `systemd-boot` · `dm-verity` · `LUKS2` · `TPM2` · `systemd-sysupdate` · `systemd-nspawn` · `Flatpak` · `Rust`

<sub>The OS repository is private until v1.0.0 · MIT license</sub>
