<div align="center">

# Hensu

### Terraform for AI Agents.

[![License](https://img.shields.io/badge/License-Apache%202.0-636366?style=flat-square&labelColor=21262d)](https://opensource.org/licenses/Apache-2.0)
[![Java](https://img.shields.io/badge/Java-25-636366?style=flat-square&logo=openjdk&logoColor=white&labelColor=21262d)](https://jdk.java.net/)
[![Protocol](https://img.shields.io/badge/Protocol-MCP-636366?style=flat-square&labelColor=21262d)](https://modelcontextprotocol.io/)
[![Status](https://img.shields.io/badge/Status-Pre--Beta-FF9F0A?style=flat-square&labelColor=21262d)]()

</div>

---

Most AI workflow frameworks treat the graph as a library concern, wired directly into application
code. Whether the workflow is any good becomes clear only after the integration work, and by then
the AI logic and the application are entangled.

Hensu applies the Terraform pattern instead. A workflow is declared as a directed graph in a
type-safe Kotlin DSL, compiled to a portable JSON artifact, and executed by a dedicated engine. Run
it locally with real agents during development, then push the same artifact to the server. The
engine is identical in both places, so there is no integration gap.

One engine, two runtimes with deliberately different powers: a server that executes nothing
locally, and a CLI that executes exactly what you granted it, under an OS sandbox.

---

## What a Workflow Looks Like

Three parallel review branches, majority-vote consensus, and a bounded revise loop that fails
cleanly instead of spinning.

```kotlin
fun contentPipeline() = workflow("content-pipeline") {
    agents {
        agent("writer")   { role = "Content Writer";   model = Models.CLAUDE_HAIKU_4_5 }
        agent("reviewer") { role = "Content Reviewer"; model = Models.GEMINI_3_1_PRO }
    }
    state {
        input("topic", VarType.STRING)
        variable("draft", VarType.STRING, "the full written article text")
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
            branch("clarity")  { agent = "reviewer"; prompt = "Review for clarity: {draft}" }
            consensus { strategy = ConsensusStrategy.MAJORITY_VOTE }
            onConsensus   goto "done"
            onNoConsensus revise "write" retry 3 otherwise "needs-work"   // feedback travels back
        }
        end("done", ExitStatus.SUCCESS)
        end("needs-work", ExitStatus.FAILURE)
    }
}
```

---

## Modules

| Module                                                                                    | Role                                                                                                        |
|:------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------|
| **[hensu-dsl](https://github.com/hensu-project/hensu/tree/main/hensu-dsl)**               | Type-safe Kotlin DSL – compiles `.kt` workflow definitions into portable JSON artifacts                     |
| **[hensu-core](https://github.com/hensu-project/hensu/tree/main/hensu-core)**             | Zero-dependency Java engine – state transitions, rubric evaluation, agent orchestration                     |
| **[hensu-cli](https://github.com/hensu-project/hensu/tree/main/hensu-cli)**               | Local runtime – `run`, `build`, `push`, `attach`; runs catalog commands under an OS sandbox; warm daemon    |
| **[hensu-server](https://github.com/hensu-project/hensu/tree/main/hensu-server)**         | Multi-tenant GraalVM native server – MCP split-pipe routes every tool call to tenant clients                |
| **[hensu-mcp](https://github.com/hensu-project/hensu/tree/main/hensu-mcp)**               | Runtime-agnostic MCP protocol layer – JSON-RPC messages, schema conversion, result rendering                |

---

## Security

- **The server executes nothing.** It is a pure orchestrator: no shell, no `eval`, no script
  runner. Tool calls travel over an outbound SSE split-pipe to tenant clients, so there are no
  inbound ports and no firewall rules on the client side. It ships as a GraalVM native image.
- **The CLI executes what you granted.** `commands.yaml` is the allowlist: agents pass a command id
  and parameter values, never command text. Every command launches under bubblewrap (Linux) or
  Seatbelt (macOS), closed by default, and a host with no working sandbox refuses the command rather
  than running it unsandboxed.

---

<div align="center">

[Monorepo](https://github.com/hensu-project/hensu) · [Get Started](https://github.com/hensu-project/hensu#getting-started) · [DSL Reference](https://github.com/hensu-project/hensu/blob/main/docs/dsl-reference.md) · [Architecture](https://github.com/hensu-project/hensu/blob/main/docs/unified-architecture.md) · [Spring Reference Client](https://github.com/hensu-project/hensu/tree/main/integrations/spring-reference-client)

---

Java 25 · Kotlin DSL · Quarkus · GraalVM Native Image · MCP

<sub>Hensu™ and the axolotl logo are trademarks of Aleksandr Suvorov.<br>
Copyright 2025–2026 Aleksandr Suvorov. All rights reserved.</sub>

</div>
