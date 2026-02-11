<div align="center">

# Hensu

### The orchestration engine for declarative AI workflows.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Java](https://img.shields.io/badge/Java-25-ED8B00?logo=openjdk&logoColor=white)](https://jdk.java.net/)
[![Build](https://img.shields.io/badge/Build-Native%20Image-success)](https://graalvm.org/)
[![Protocol](https://img.shields.io/badge/Protocol-MCP-green)](https://modelcontextprotocol.io/)
[![Status](https://img.shields.io/badge/Status-Alpha-red)]()

**Define. Build. Run.**<br>
Self-hosted. Developer-friendly. Zero lock-in.

</div>

---

Shipping multi-agent systems today means scattering orchestration logic across application code, making it impossible to
version, test, or deploy agent behavior independently. **Hensu** treats workflows as **compilable artifacts** — authored
in a type-safe Kotlin DSL, compiled to portable JSON, and executed on a stateless native-image server that never runs
user code. Think **Terraform for AI Agents**.

```kotlin
fun contentPipeline() = workflow("ContentPipeline") {
    agents {
        agent("writer") { role = "Content Writer"; model = Models.CLAUDE_SONNET_4_5 }
        agent("reviewer") { role = "Content Reviewer"; model = Models.GPT_4O }
    }

    graph {
        start at "write"

        node("write") {
            agent = "writer"
            prompt = "Write an article about {topic}"
            onSuccess goto "review"
        }

        parallel("review") {
            branch("quality") { agent = "reviewer"; prompt = "Review for quality: {write}" }
            branch("accuracy") { agent = "reviewer"; prompt = "Review for accuracy: {write}" }

            consensus { strategy = ConsensusStrategy.MAJORITY_VOTE }
            onConsensus goto "done"
            onNoConsensus goto "write"   // self-correcting loop
        }

        end("done", ExitStatus.SUCCESS)
    }
}
```

---

## Why Hensu

| Problem                                              | Hensu's Answer                                                                                          |
|:-----------------------------------------------------|:--------------------------------------------------------------------------------------------------------|
| Workflow logic is buried in application code         | **Declarative DSL** compiles to versioned JSON artifacts — diff, review, and deploy like infrastructure |
| No way to test agent orchestration without API calls | **Stub agents** and a pure-Java core enable full offline testing                                        |
| Multi-agent coordination requires custom glue code   | **Parallel branches**, **consensus strategies**, and **sub-workflows** are first-class primitives       |
| Quality is checked after the fact                    | **Rubric evaluation** gates node transitions — fail fast, not at the end                                |
| Production deployments need human oversight          | **Checkpoints** pause execution for manual approval before high-stakes transitions                      |
| Vendor lock-in to a specific LLM provider            | **Model-agnostic agents** — use Claude, GPT, Gemini, or any provider in the same workflow               |

---

## Architecture at a Glance

```text
 Developer (local)                                               Hensu Runtime              External
 +----------+    +----------+    +----------+    +----------+    +---------------------+    +--------------+
 | Kotlin   |    |  hensu   |    |   JSON   |    |  hensu   |    |  Hensu Server       |    | LLMs (Claude |
 | DSL      |--->|  build   |--->|   Def.   |--->|  push    |--->|  (GraalVM Native)   |<-->| GPT, Gemini) |
 +----------+    +----------+    +----------+    +----------+    |                     |    +--------------+
                                                                 |  Core Engine        |    +--------------+
                                                                 |  +- State Manager   |<-->| MCP Tool     |
                                                                 |  +- Rubric Evaluator|    | Servers      |
                                                                 |  +- Consensus Engine|    +--------------+
                                                                 +---------------------+
```

**The Hensu Stack:**

| Module                                                                                          | Role                                                                                              |
|:------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|
| **[hensu-dsl](https://github.com/hensu-project/hensu/tree/main/hensu-dsl)**                     | Type-safe Kotlin DSL that compiles workflows into portable JSON definitions                       |
| **[hensu-core](https://github.com/hensu-project/hensu/tree/main/hensu-core)**                   | Zero-dependency Java execution engine — state transitions, rubric evaluation, agent orchestration |
| **[hensu-server](https://github.com/hensu-project/hensu/tree/main/hensu-server)**               | Stateless multi-tenant server with MCP Split-Pipe transport for secure remote tool execution      |
| **[hensu-serialization](https://github.com/hensu-project/hensu/tree/main/hensu-serialization)** | Jackson mixins that bridge in-memory domain objects and their JSON wire format                    |
| **[hensu-cli](https://github.com/hensu-project/hensu/tree/main/hensu-cli)**                     | Developer CLI — `build`, `push`, `pull`, `list`, `delete` workflows on remote servers             |

---

## Security Model

The server is a **pure orchestrator** — it never executes user-supplied code.

- **No Local Execution.** No shell, no `eval`, no script runner. All side effects route through MCP to tenant clients.
- **Tenant Isolation.** Every execution runs inside a Java `ScopedValue` boundary. No data leaks between concurrent
  workflows.
- **No Inbound Ports.** The Split-Pipe transport means tenant clients connect *outbound* via SSE. No firewall rules
  required.
- **Native Binary.** GraalVM native image eliminates classpath scanning, reflection, and dynamic class loading attack
  surfaces.

---

## Project Status

Hensu is in **Alpha**. The core architecture is stable and the engine is feature-complete. Current focus is on
comprehensive integration testing and persistence layer hardening.

- [x] Core engine implementation (state machine, rubric evaluation, consensus, sub-workflows)
- [x] Type-safe Kotlin DSL with compile-time validation
- [x] CLI for workflow lifecycle management
- [x] Server prototype with MCP Split-Pipe transport
- [x] Integration test suite (standard nodes, parallel, planning, review, rubric, pause/resume)
- [ ] Persistence layer (database integration)
- [ ] Production security hardening
- [ ] Performance benchmarks and scaling documentation

---

## Get Started

```shell
git clone https://github.com/hensu-project/hensu.git && cd hensu
./gradlew build -x test
java -jar hensu-server/build/quarkus-app/quarkus-run.jar
```

**Resources:**

- **[Hensu Monorepo](https://github.com/hensu-project/hensu)** — Engine, Server, DSL, CLI, and Serialization
- **[DSL Reference](https://github.com/hensu-project/hensu/blob/master/docs/dsl-reference.md)** — Complete guide to
  building agentic workflows
- **[Architecture](https://github.com/hensu-project/hensu/blob/master/docs/unified-architecture.md)** — Design
  decisions, module structure, execution model, and security architecture

---

<div align="center">

Built with Java 25 · Kotlin · Quarkus · GraalVM Native Image · MCP Protocol

</div>
