# Hermatic — Roadmap

> Solo developer, part-time, ~12 focused hours/week.
> Target pace: ~9–14 months to v1.0.0. Don't fight the pace — build around it.

---

## Operating principles

These are the rules that make this roadmap survive contact with reality.

- **One session = one resumable unit.** Every checkbox below should fit in a single 60–90 min block. If it doesn't, split it before starting.
- **End every session with a "next step" note.** Write the literal next command or file you'd open. Tired-you tomorrow won't remember.
- **Commit often, even broken.** Use WIP branches. Your working memory is the git log.
- **Boring infrastructure first.** A reproducible build + a fast QEMU loop pays back every session after.
- **Cut scope, never rigor.** Drop features (e.g. sysext → monolithic), never drop the verity/signing/reproducibility guarantees.
- **Don't context-switch phases.** Finish the current phase's exit criteria before touching the next.

---

## Phase 0 — MKOSI baseline

**Goal:** `mkosi build` produces a UKI that boots in QEMU/OVMF into a read-only dm-verity rootfs with a minimal systemd userspace.

**Estimated calendar time:** 6–8 weeks at 12 h/week.

**Exit criteria:**
- [ ] `mkosi build && mkosi qemu` boots to a login prompt
- [ ] Root is read-only and verity-protected (verify by trying to write — should fail)
- [ ] `systemctl list-units --state=running` shows only the intended units
- [ ] Build is reproducible: two clean builds produce identical image hashes

### 0.1 — Environment & repo setup
- [ ] Repo initialized, license chosen, README stub
- [ ] MKOSI ≥24 installed and version-pinned in a dev container or nix shell
- [ ] OVMF firmware located and documented in README
- [ ] `swtpm` installed for later phases (don't wire it in yet)
- [ ] First commit: empty `mkosi.conf` that builds *something*, even if trivial

### 0.2 — Minimal bootable image
- [ ] `mkosi.conf` produces a Linux + systemd image (no verity, no UKI yet)
- [ ] Image boots in QEMU with `mkosi qemu`
- [ ] Document the QEMU invocation in a Makefile or `justfile` so it's one command

### 0.3 — UKI assembly
- [ ] Switch boot path to systemd-boot + systemd-stub
- [ ] ukify produces a signed-able UKI (self-signed test keys for now)
- [ ] Image boots from UKI in QEMU

### 0.4 — dm-verity rootfs
- [ ] `Format=disk` + `Verity=signed` in `mkosi.conf`
- [ ] Verity root hash embedded in UKI kernel cmdline
- [ ] Boots into verity-protected root
- [ ] Attempted write to `/` fails as expected

### 0.5 — Userspace strip
- [ ] Identify every unit running by default; document why each exists or remove it
- [ ] Final set: `systemd-networkd`, `systemd-resolved`, `systemd-journald`, plus core systemd
- [ ] Login works; `journalctl` works; network works inside QEMU

### 0.6 — Reproducibility check
- [ ] Clean build twice, diff the image bytes (or hashes)
- [ ] If they differ, track down the source of nondeterminism before moving on
- [ ] Document any unavoidable nondeterminism with workarounds

---

## Phase 1 — State and config

**Goal:** A declarative TOML config applies idempotently on boot. Workloads run as nspawn containers on the immutable base.

**Estimated calendar time:** 10–12 weeks at 12 h/week.

**Exit criteria:**
- [ ] A TOML file declares hostname, network, users, enabled units, kernel params
- [ ] Booting with that config produces the declared state; booting again is a no-op
- [ ] At least one nspawn workload starts via `machinectl` from `/var/lib/machines/`
- [ ] An nspawn image is also produced by MKOSI from a sibling config

### 1.1 — Config schema design
- [ ] First-draft TOML schema written as a spec document (not code yet)
- [ ] Schema reviewed against the four guiding principles — anything mutable or non-idempotent gets cut
- [ ] Versioned (`schema_version = 1`) from day one

### 1.2 — CLI scaffold (Rust or Go — pick now)
- [ ] Language picked and documented with reasoning (1 paragraph)
- [ ] `marrow` (or `hermatic`) binary builds, has `--help`, has `status` returning placeholder JSON
- [ ] Binary is statically linked or has explicit runtime deps documented

### 1.3 — Config parsing & validation
- [ ] Parse TOML into typed structs
- [ ] Validate: reject unknown keys, enforce required fields, version check
- [ ] Round-trip test: parse → re-emit → re-parse equals original

### 1.4 — Apply logic (idempotent)
- [ ] `hermatic apply <config.toml>` renders hostname, network units, user definitions into the correct locations
- [ ] Re-running apply produces no changes (verify with `journalctl` and file mtimes)
- [ ] Apply on boot via a systemd unit ordered before `multi-user.target`

### 1.5 — Nspawn workload layer
- [ ] Second MKOSI config that builds a minimal nspawn machine image
- [ ] Machine image installed to `/var/lib/machines/` at build time
- [ ] `.nspawn` unit file declares the workload
- [ ] `machinectl start <name>` brings it up; networkd handles its veth

### 1.6 — Sysext (with fallback gate)
- [ ] Spike: get one systemd-sysext layer mounting into `/usr` (timebox: 2 weeks of sessions)
- [ ] **Decision point:** if sysext is fighting back after the timebox, fall back to monolithic per-role images and document the decision
- [ ] Whichever path: extension/role layering produces a measurable change at runtime

---

## Phase 2 — Atomic updates (gate to bare metal)

**Goal:** Update in QEMU, reboot into new image, verify automatic rollback on failed boot. Must be rock-solid before any hardware work.

**Estimated calendar time:** 8–10 weeks at 12 h/week.

**Exit criteria:**
- [ ] Two slots (root-a, root-b) provisioned via systemd-repart from a declarative config
- [ ] `hermatic update` stages a new image into the inactive slot, verifies signature, switches boot order
- [ ] Successful boot promotes the new slot; failed boot rolls back automatically via boot counter
- [ ] `hermatic rollback` manually reverts to the previous slot
- [ ] All of the above survives a 50-cycle automated soak test in QEMU

### 2.1 — Partition layout
- [ ] DPS-compliant partition definitions in systemd-repart config
- [ ] First boot from blank disk provisions ESP, root-a, root-b, /var
- [ ] Document the layout in a diagram (ASCII is fine)

### 2.2 — Sysupdate plumbing
- [ ] systemd-sysupdate config that fetches/validates a new image artifact
- [ ] Local file-based transfer first (no HTTP server yet — keep it offline)
- [ ] Signature verification using the same keys as UKI signing

### 2.3 — Slot switching
- [ ] systemd-boot picks the staged slot on next boot
- [ ] Boot counter decrements on each attempt; reaches zero → automatic rollback
- [ ] First successful boot of new slot clears the counter and marks it good

### 2.4 — CLI surface (first cut)
- [ ] `hermatic update <image>` — stages, verifies, switches
- [ ] `hermatic rollback` — flips to the previous slot
- [ ] `hermatic status` — current slot, version, verity hash, last update

### 2.5 — Soak test
- [ ] Scripted QEMU harness that: boots, updates, reboots, verifies, repeats
- [ ] Inject failures: bad signature, truncated image, kernel panic on new slot, power-cut mid-update
- [ ] 50 clean cycles + every failure mode rolls back correctly

---

## Phase 3 — Trust and attestation

**Goal:** The system attests its own boot chain. Secrets only unseal when measured state matches.

**Estimated calendar time:** 8 weeks at 12 h/week.

**Exit criteria:**
- [ ] Secure Boot enabled in QEMU with custom enrolled keys
- [ ] UKIs signed at build time, rejected by firmware if tampered
- [ ] TPM2 (swtpm) measurements drive systemd-pcrlock policy
- [ ] A test credential unseals only when PCRs match the expected boot state

### 3.1 — Secure Boot in QEMU
- [ ] Custom PK/KEK/db keys generated via `sbctl`
- [ ] Keys enrolled in OVMF NVRAM
- [ ] Unsigned UKI rejected; signed UKI accepted

### 3.2 — TPM2 with swtpm
- [ ] swtpm attached to QEMU; `tpm2_pcrread` works inside guest
- [ ] PCR values reproducible across boots of the same image

### 3.3 — pcrlock policy
- [ ] **Pre-check:** confirm systemd-pcrlock version & stability in your target systemd version before building on it
- [ ] Generate a pcrlock policy from a known-good boot
- [ ] Seal a test credential against the policy
- [ ] Unseal succeeds on matching boot, fails on tampered boot

### 3.4 — systemd-creds delivery
- [ ] One nspawn workload receives an encrypted credential via `LoadCredentialEncrypted=`
- [ ] Credential is not present anywhere on disk in plaintext

---

## Phase 4 — v1.0.0

**Estimated calendar time:** 4–5 weeks at 12 h/week.

**Exit criteria:**
- [ ] Full CLI: `apply`, `update`, `rollback`, `verify`, `status` — all with `--help`
- [ ] Reproducibility audit passes (two builds, identical hashes)
- [ ] Signed release image published with SLSA-style provenance
- [ ] Docs: install, first boot, config reference, update flow, rollback flow

### 4.1 — `verify` command
- [ ] Checks: verity hash, UKI signature, slot integrity, pcrlock state
- [ ] Returns structured output (JSON) and a human-readable summary

### 4.2 — Reproducibility audit
- [ ] Clean-room build on a second machine matches your build hash
- [ ] If it doesn't, fix before tagging

### 4.3 — Docs pass
- [ ] README: what Hermatic is, who it's for, what it isn't
- [ ] Install guide (QEMU + bare metal)
- [ ] Config reference (autogenerated from schema if possible)
- [ ] Architecture doc (one diagram, one page of prose)

### 4.4 — Release
- [ ] Tag `v1.0.0`, sign the release image
- [ ] Publish provenance and verity root hashes
- [ ] Announcement post drafted (optional — only if you want users)

---

## Bare-metal transition (between Phase 2 and Phase 3)

Additive only. Architecture does not change.

- [ ] swtpm → real TPM2 chip; re-run Phase 3 setup against it
- [ ] Secure Boot keys enrolled in firmware (vendor-specific dance — document per machine)
- [ ] Provisioning path: MKOSI ISO output OR iPXE — pick one for v1
- [ ] One physical machine boots, updates, rolls back end-to-end

---

## Risks — keep visible

- **A/B rollback edge cases** — the single hardest piece. Soak test obsessively.
- **systemd-sysext** — sparse docs, sharp edges. Fallback to monolithic role images is pre-approved.
- **systemd-pcrlock** — very new. Verify version before building on it (Phase 3.3 pre-check).
- **systemd-sysupdate** — less battle-tested than the rest. Lean on the QEMU soak test.
- **Solo-parent reality** — illness, regressions, no-sleep weeks. Build slack into estimates, not optimism.

---

## Session log

> Append a line at the end of every session: date, what got done, the literal next step.

- _YYYY-MM-DD_ — _what you did_ — **Next:** _exact next action_
