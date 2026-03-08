<div align="center">

# Hensu™

### Terraform for AI Agents.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Java](https://img.shields.io/badge/Java-25-ED8B00?logo=openjdk&logoColor=white)](https://jdk.java.net/)
[![CI](https://github.com/hensu-project/hensu/actions/workflows/ci.yml/badge.svg)](https://github.com/hensu-project/hensu/actions/workflows/ci.yml)
[![Native Image](https://github.com/hensu-project/hensu/actions/workflows/native.yml/badge.svg)](https://github.com/hensu-project/hensu/actions/workflows/native.yml)
[![Protocol](https://img.shields.io/badge/Protocol-MCP-green)](https://modelcontextprotocol.io/)

</div>

---

Shipping multi-agent systems today means weeks of SDK boilerplate, Python GIL limits, or AI logic tangled
deep inside application code — before you know if the workflow is even useful. Then you rewrite everything.

**Hensu** ends that cycle. Author a workflow in a type-safe Kotlin DSL. Validate the graph locally with
zero-cost stub agents. Switch to real agents on the same engine. Push the same compiled artifact to the
server. Integration is done — no rework, no coupling, no rewrite.

---

## Why Hensu?

| Alternative                           | The Cost                                                                                                                         | Hensu's Answer                                                                                                   |
|:--------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| **Temporal / Camunda / Airflow**      | Weeks of worker setup, SDK boilerplate, and serialization contracts before a single line of business logic runs.                 | Install the CLI, run a stub workflow in minutes. Same artifact goes to production — no integration rework.       |
| **LangGraph / CrewAI / AutoGen**      | Python GIL kills true concurrency. `asyncio` is cooperative, not parallel. AI logic couples tightly to application code.         | Java 25 virtual threads. Real parallel execution. Workflows are standalone artifacts — decoupled from your app.  |
| **LangChain4J / Spring AI / Embabel** | Forces graph creation directly into your codebase. Weeks of integration to discover whether the workflow is even useful.         | DSL compiles to an independent artifact. Test the graph via CLI in minutes, without touching your core codebase. |
| **Custom in-house orchestrators**     | Inevitably mix orchestration with tool execution, creating a remote-code-execution surface. Rewrites happen when AI logic grows. | Hard security boundary via MCP split-pipe. Orchestrators only orchestrate. Tools run where you control them.     |

---

## The DSL

Two agents, parallel review branches, majority-vote consensus, self-correcting loop — in under 30 lines:

```kotlin
fun contentPipeline() = workflow("content-pipeline") {
    agents {
        agent("writer")   { role = "Content Writer";   model = Models.CLAUDE_SONNET_4_5 }
        agent("reviewer") { role = "Content Reviewer"; model = Models.GPT_4O }
    }
    state {
        input("topic",  VarType.STRING)
        variable("draft", VarType.STRING)
    }
    graph {
        start at "write"
        node("write") {
            agent  = "writer"
            prompt = "Write an article about {topic}."
            writes("draft")
            onSuccess goto "review"
        }
        parallel("review") {
            branch("quality")  { agent = "reviewer"; prompt = "Review for quality: {draft}" }
            branch("accuracy") { agent = "reviewer"; prompt = "Review for accuracy: {draft}" }
            consensus { strategy = ConsensusStrategy.MAJORITY_VOTE }
            onConsensus   goto "done"
            onNoConsensus goto "write"
        }
        end("done", ExitStatus.SUCCESS)
    }
}
```

---

## Architecture

```
 Developer (local)                                              Hensu Runtime               External

 Develop:   +——————————+    +——————————+
            │ Kotlin   │———>│  hensu   │   (stubs first → real agents; same hensu-core as the server)
            │ DSL      │    │  run     │
            +——————————+    +——————————+

 Deploy:    +——————————+    +——————————+    +——————————+    +——————————————————————+    +——————————————+
            │  hensu   │    │   JSON   │    │  hensu   │    │  Hensu Server        │    │ LLMs (Claude │
            │  build   │———>│   Def.   │———>│  push    │———>│  (GraalVM Native)    │<——>│ GPT, Gemini) │
            +——————————+    +——————————+    +——————————+    │                      │    +——————————————+
                                                            │  Core Engine         │    +——————————————+
                                                            │  +— State Manager    │<——>│ MCP Tool     │
                                                            │  +— Rubric Evaluator │    │ Servers      │
                                                            │  +— Consensus Engine │    +——————————————+
                                                            +——————————————————————+
```

**The stack:**

| Module                                                                            | Role                                                                                              |
|:----------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|
| **[hensu-dsl](https://github.com/hensu-project/hensu/tree/main/hensu-dsl)**       | Type-safe Kotlin DSL — compiles `.kt` workflow definitions into portable JSON artifacts           |
| **[hensu-core](https://github.com/hensu-project/hensu/tree/main/hensu-core)**     | Zero-dependency Java execution engine — state transitions, rubric evaluation, agent orchestration |
| **[hensu-server](https://github.com/hensu-project/hensu/tree/main/hensu-server)** | Multi-tenant GraalVM native server — MCP split-pipe transport for secure remote tool execution    |
| **[hensu-cli](https://github.com/hensu-project/hensu/tree/main/hensu-cli)**       | Developer CLI — `run`, `build`, `push`, `pull`, `list`, `attach`; local daemon keeps the JVM warm |

---

## Security Model

The server is a **pure orchestrator** — it never executes user-supplied code.

- **No local execution.** No shell, no `eval`, no script runner. Side effects route through MCP to tenant clients — a hallucinating LLM cannot run code on the orchestrator.
- **No inbound ports.** Tenant clients connect *outbound* via SSE. No firewall rules, no VPN.
- **Tenant isolation.** Every execution runs inside a Java `ScopedValue` boundary — no data leaks between concurrent workflows.
- **Native binary.** GraalVM native image eliminates classpath scanning, reflection, and dynamic class loading attack surfaces.
- **API boundary.** JWT authentication, safe-identifier validation, control-character filtering, and LLM output sanitization at every entry point.

---

**[→ Monorepo](https://github.com/hensu-project/hensu)** · [DSL Reference](https://github.com/hensu-project/hensu/blob/main/docs/dsl-reference.md) · [Architecture](https://github.com/hensu-project/hensu/blob/main/docs/unified-architecture.md) · [Spring Reference Client](https://github.com/hensu-project/hensu/tree/main/integrations/spring-reference-client)

---

<div align="center">

Java 25 · Kotlin · Quarkus · GraalVM Native Image · MCP Protocol

</div>
