![Hermatic — the immutable operating substrate](https://raw.githubusercontent.com/hermatic/.github/main/profile/futuristic_branding_banner.png)

<br>

<img src="https://raw.githubusercontent.com/hermatic/.github/main/profile/hermatic_github_avatar.png" width="48" align="left" style="margin-right: 14px">

**HERMATIC**
`the immutable operating substrate`

&nbsp;

![phase 1.6](https://img.shields.io/badge/phase-1.6-7b5ea7?style=flat-square&labelColor=1a1a1f)
![active dev](https://img.shields.io/badge/status-active%20dev-639922?style=flat-square&labelColor=1a1a1f)
![not ready for use](https://img.shields.io/badge/⚠-not%20ready%20for%20use-a89840?style=flat-square&labelColor=1a1a1f)
![MIT](https://img.shields.io/badge/license-MIT-555?style=flat-square&labelColor=1a1a1f)

---

## *Systems without drift.*

An image-based Linux OS built around a hermetic, immutable `/usr/`. dm-verity on every read. TPM2-bound secrets. Reproducible builds. Atomic A/B updates.

Inspired by Poettering's ["Fitting Everything Together"](https://0pointer.net/blog/fitting-everything-together.html) — built from real distribution packages, deployed as signed images.

---

## Core properties

| | |
|---|---|
| **Immutable core** | A verified foundation. Minimal. Permanent. |
| **Composable layers** | Deterministic extensions. Reproducible. Declarative. |
| **Isolated runtime** | Controlled execution environments via nspawn. |
| **Verification** | Measured. Attested. Cryptographically verified. |

---

## Trust chain

```
firmware → systemd-boot → UKI → dm-verity → LUKS2 + TPM2 → userspace
   │              │         │         │             │              │
UEFI validates  picks    signed PE  /usr/ hash   root unseals   hermatic
 boot loader   newest    kernel +   pinned in    on measured    apply →
               UKI       initrd +   cmdline      boot state     workloads
                         cmdline
```

---

## What it does today

| | |
|---|---|
| `boot` | UKI — kernel + initrd + cmdline, signed as one PE binary |
| `root` | hermetic `/usr/` verity-protected, read-only |
| `state` | `/etc` and `/var` on a separate writable partition |
| `config` | `hermatic.toml` — declarative, idempotent, applied on every boot |
| `workloads` | nspawn containers, systemd PID 1, private veth network |
| `debug` | sysext overlay — `strace` / `gdb` / `ltrace` on demand without modifying the base image |
| `builds` | reproducible — two clean builds from the same source produce identical image hashes |
| `tpm2` | present and measuring PCRs at boot |
| `cli` | `hermatic apply` · `parse` · `status` (Rust, statically linked musl) |

---

## Roadmap

```
✓  Phase 0   MKOSI baseline — UKI boots into verity rootfs in QEMU
✓  Phase 1   Declarative TOML config, hermatic CLI, nspawn workloads, sysext debug overlay
→  Phase 2   A/B atomic updates with automatic rollback                         ← you are here
·  Phase 3   Secure Boot key enrollment, TPM2 attestation, pcrlock policy
·  v1.0.0   Full CLI, reproducibility audit, signed release, provenance
```

---

## Design principles

1. **Images, not packages** at runtime. Packages are a build-time tool.
2. **Every read verified.** dm-verity on `/usr/` — not just at mount time.
3. **Secrets sealed to boot state.** TPM2 binds keys to the measured chain.
4. **Updates atomic, rollback automatic.** Boot counter + A/B slots.
5. **Factory reset is a reboot.** Erase the writable partition — done.
6. **Reproducible.** Identical inputs produce identical image bytes.
7. **Cut scope, never rigor.** Drop features; never drop the guarantees.

---

## Repository

| Repo | Description |
|---|---|
| [hermatic/hermatic](https://github.com/hermatic/hermatic) | OS image, CLI, mkosi config, partition definitions |

---

## Stack

`mkosi` · `systemd-boot` · `ukify` · `dm-verity` · `LUKS2` · `systemd-repart` · `systemd-sysupdate` · `systemd-nspawn` · `systemd-sysext` · `systemd-creds` · `TPM2` · `Rust`

---

<sub>Solo dev · part-time · ~12 h/week · MIT license · inspired by <a href="https://0pointer.net/blog/fitting-everything-together.html">Lennart Poettering</a></sub>
