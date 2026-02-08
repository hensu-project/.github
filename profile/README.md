<div align="center">

# Hensu

### Open-Source Infrastructure for AI Agent Workflows

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Java](https://img.shields.io/badge/Java-25-ED8B00?logo=openjdk&logoColor=white)](https://jdk.java.net/)
[![Build](https://img.shields.io/badge/Build-Native%20Image-success)](https://graalvm.org/)
[![Protocol](https://img.shields.io/badge/Protocol-MCP-green)](https://modelcontextprotocol.io/)
[![Status](https://img.shields.io/badge/Status-Alpha-red)]()

**Define. Build. Run.**<br>
Self-hosted. Developer-friendly. Zero lock-in.

</div>

---

**Hensu** is a modular infrastructure stack designed to solve the "Agent-in-Production" problem. We strictly separate
the **Definition of Intelligence** from its **Execution**, providing a toolchain that feels like **Terraform for AI
Agents**.

## Core Philosophy: Separation of Intent

Hensu is built on the principle of **Describe, don't implement.** Rather than requiring imperative "wiring" to integrate
a framework's stack, Hensu utilizes a type-safe Kotlin DSL to decouple workflow logic from the underlying runtime.

* **Declarative Orchestration:** Define complex multi-agent interactions without managing internal state transitions or
  boilerplate integration code.
* **Type-Safety by Design:** Catch orchestration errors at compile-time via the Kotlin DSL.
* **Low-Friction Integration:** No need to hard-code against internal APIs; the DSL serves as the source of truth for
  agent behavior.

**Example: Defining a Workflow**

```kotlin
// Describe, don't implement.
fun myWorkflow() = workflow("weather-forecast") {
    agents {
        agent("meteorologist") {
            role = "A weather assistant"
            model = "gemini-3-flash"
            tools = listOf("geo-locator")
        }
    }

    graph {
        start at "generate"

        node("generate") {
            agent = "meteorologist"
            prompt = """
                Generate a short message about the weather today.
                Keep it to one sentence.
            """.trimIndent()
            onSuccess goto "notify"
        }

        action("notify") {
            send("slack")

            onSuccess goto "exit"
            onFailure retry 2 otherwise "exit"
        }

        end("exit")
    }
}
```

---

## Technical Architecture

Hensu is designed for performance and extensibility, utilizing a modern JVM stack to bridge declarative definitions with
scalable execution.

**High-Level Components**

- DSL Layer (Kotlin): A type-safe syntax for defining agents, roles, and communication protocols.
- CLI: The developer interface for compiling, testing, and managing workflows on remote servers.
- Core Engine: A pure Java execution runtime with zero external dependencies.
- Server: A GraalVM-native, multi-tenant environment featuring an MCP-first (Model Context Protocol) architecture.

**Architectural Philosophy**

The system is built to be entirely stateless and reactive. By decoupling execution logic from the state, Hensu
ensures horizontal scalability, near-instant startup times via GraalVM, and native interoperability with the evolving AI
tool ecosystem.

Hensu is built on a **feature-driven architecture** ensuring the foundation is natively built for high-level
orchestration, not just basic tasks.

**Hensu CLI: Developer Experience**

The CLI manages the entire lifecycle of a workflow, ensuring that what you describe is exactly what the server executes.

- Compile: Translates .kt workflow definitions into portable JSON artifacts.
- Local Execution: Runs and validates workflows locally with full agent support before deployment.
- Build & Push: Compiles and uploads workflow artifacts directly to the remote server.
- Server Management: A complete interface to push, pull, list, or delete workflows on remote instances.

---

## Project Status: Stabilization & Validation

Hensu is currently in its Alpha (Feature-Complete) phase. The architecture is stable and designed to be
production-ready; our current focus is on final verification and hardening.

**Current State:**

- Feature Maturity: The core execution logic, DSL, and CLI are fully implemented. Server is prototyped.
- Verification: Core logic is currently ~70% verified through initial testing.
- Stabilization: We are currently focused on comprehensive integration testing to ensure the thoughtfully designed
  architecture handles all edge cases.

**Roadmap to Production**

```
[x] Feature-Driven Architecture Design
[x] Core DSL Syntax Definition
[x] Engine Implementation
[x] CLI Implementation
[x] Server Prototype
[ ] (Current) Full Integration Test Suite
[ ] Persistence Layer (Database Integration)
[ ] Production Security Hardening
[ ] Performance Tuning & Scaling Docs
```

## Getting Started with Hensu

* **[Hensu Monorepo](https://github.com/hensu-project/hensu)**: The core infrastructure containing the Engine, Server,
  and CLI.
* **[DSL Reference](https://github.com/hensu-project/hensu/tree/main/docs/dsl-reference.md)**: Documentation on building
  agentic workflows.
* **[Community & Contributing](../CONTRIBUTING.md)**: Join us in building the
  future of agentic infrastructure.

<div align="center">
Built with Java 25 • Kotlin • Quarkus • GraalVM • MCP
</div>
