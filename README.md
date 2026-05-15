# HERMATIC

Immutable operating infrastructure.

HERMATIC is an image-based operating substrate focused on:
- deterministic systems,
- declarative state,
- measured boot,
- signed artifacts,
- and reproducible deployments.

The system is designed to minimize drift, reduce operational entropy, and provide a trusted foundation for modern infrastructure workloads.

---

## Principles

- Immutable by default
- Atomic system updates
- Declarative configuration
- Trusted execution paths
- Minimal operational surface
- Reproducible state

---

## Architecture

HERMATIC treats the operating system as a verified and reproducible artifact rather than a mutable runtime environment.

Core concepts include:
- image-based deployments,
- measured boot,
- TPM-backed trust,
- atomic upgrades,
- and isolated workload execution.

---

## Philosophy

Everything intentional.

HERMATIC reduces the operating environment to its essential structure:
a stable, measurable, and reproducible substrate for systems that must remain predictable over time.

State drift is treated as a liability.

---

## Status

HERMATIC is currently under active development.

Early repositories and architecture components will appear here as the platform evolves.

---

## Planned Components

| Component | Purpose |
|---|---|
| `hermatic-core` | Base operating image |
| `hermatic-boot` | Boot and trust chain |
| `hermatic-update` | Atomic image updates |
| `hermatic-image` | Image generation tooling |
| `hermatic-runtime` | Workload execution environment |
| `hermaticctl` | System management CLI |

---

## Design Goals

- Minimal host systems
- Deterministic infrastructure
- Strong trust boundaries
- Operational simplicity
- Long-term maintainability
- Cohesive system architecture

---

## Inspiration

HERMATIC exists in the space between:
- :contentReference[oaicite:0]{index=0}
- :contentReference[oaicite:1]{index=1}
- :contentReference[oaicite:2]{index=2}
- and modern image-based infrastructure systems.

But with a stronger focus on cohesion, restraint, and infrastructure-grade operational design.

---

## License

Repository licenses vary by project.
See individual repositories for details.
