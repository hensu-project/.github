# Contributing to Hensu™

Thank you for your interest in contributing to Hensu! To maintain the quality of this industrial-grade infrastructure,
we follow a disciplined development process.

## Technical Foundation

- **Language:** Java 25 / Kotlin 2.x
- **Framework:** Quarkus
- **Build Tool:** Gradle (with Toolchain support)
- **Target:** Native binaries via GraalVM

---

## How to Contribute

### 1. The Pull Request Process

- **Mandatory Approval:** All PRs require an approving review from the Code Owner (@alxsuv).
- **Linear History:** We use **Squash Merges** only. Please keep your branch history clean.
- **Native Verification:** If your change affects the `dsl`, `core` or `server`, you must verify that the project still
  compiles to a GraalVM native image.

### 2. Development Standards

- **Conventional Commits:** Use `feat:`, `fix:`, `docs:`, `refactor:`, or `chore:`.
- **Documentation:** Updates to `dsl`, `core` or `server` require corresponding `/docs` updates.
- **No Reflection:** Avoid using Java reflection unless absolutely necessary for Quarkus/GraalVM compatibility.

---

## Testing & Verification

We are moving toward 100% verification. Every PR should include:

- Unit tests for new logic.
- Integration tests for DSL-to-Engine transitions.
- A confirmation that `gradle nativeCompile` succeeds.

---

## Reporting Issues

If you find a bug or a performance bottleneck, please open an issue with the `type: bug` or `area: performance` label.
Provide a minimal reproducible example, preferably as a small DSL snippet.

---

## Legal

### 1. The Legal Framework (The "Commercial-Ready" Clause)

Hensu is an open-source project licensed under **Apache 2.0**. To ensure the project's long-term sustainability and our
ability to offer enterprise support or "Pro" versions (similar to the Nginx or HashiCorp models), we use an
**Inbound=Outbound** contribution model with a specific re-licensing grant.

**By submitting a Pull Request, you agree to the following:**

* **License Grant:** You grant the Hensu project owner (and its legal successors/LLC) a perpetual, worldwide,
  non-exclusive, royalty-free, irrevocable license to use, modify, and distribute your contribution.
* **Commercial & Re-licensing Rights:** You acknowledge that the project owner reserves the right to distribute the
  project (including your contributions) under different license terms, including commercial or proprietary licenses.
  This allows us to fund the core open-source development through commercial offerings.
* **Authenticity:** You certify that you wrote the code or have the legal right to submit it.

### 2. No Signatures Required: Use Git Sign-Off (-s)

We do not want to slow you down with legal forms. Instead, we use the **Developer Certificate of Origin (DCO)**. To
"sign" your agreement to the terms above, simply add the `-s` flag to your commits:

```bash
git commit -s -m "feat: added support for virtual threads in worker nodes"
```

### 3. Trademark Policy

This project follows the trademark guidelines outlined in [TRADEMARK.md](TRADEMARK.md). Use of project trademarks
is subject to this policy.

### 4. License Compliance

All contributions must be compatible with the **Apache License 2.0**. Do not submit any code that is subject to a
license that is incompatible with Apache 2.0.

---

**Lead Developer:** [@alxsuv](https://github.com/alxsuv)
*"Building the infrastructure for the next generation of JVM agents."*
