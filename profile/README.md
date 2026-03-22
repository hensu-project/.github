<div align="center">

# Hensu

### Terraform for AI Agents.

[![License](https://img.shields.io/badge/License-Apache%202.0-636366?style=flat-square&labelColor=21262d)](https://opensource.org/licenses/Apache-2.0)
[![Java](https://img.shields.io/badge/Java-25-636366?style=flat-square&logo=openjdk&logoColor=white&labelColor=21262d)](https://jdk.java.net/)
[![Protocol](https://img.shields.io/badge/Protocol-MCP-636366?style=flat-square&labelColor=21262d)](https://modelcontextprotocol.io/)
[![Status](https://img.shields.io/badge/Status-Pre--Beta-FF9F0A?style=flat-square&labelColor=21262d)]()

</div>

---

Most AI workflow frameworks treat graph construction as a library concern – something wired directly
into application code. You discover whether the workflow is actually useful only after weeks of
integration work. By then the AI logic and the application are entangled, and neither can change
cleanly without the other.

Hensu takes a different approach. Workflows are standalone compiled artifacts, defined in a
type-safe Kotlin DSL and executed by a dedicated engine. Run a workflow locally with real agents
during development. Push the same compiled artifact to the server. The engine is identical in both
environments, so there is no integration gap between the two.

---

## DSL Example

Parallel review branches, majority-vote consensus, self-correcting loop:

```kotlin
fun contentPipeline() = workflow("content-pipeline") {
    agents {
        agent("writer")   { role = "Content Writer";   model = Models.CLAUDE_HAIKU_4_5 }
        agent("reviewer") { role = "Content Reviewer"; model = Models.GEMINI_3_1_PRO }
    }
    state {
        input("topic",  VarType.STRING)
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
            consensus { strategy = ConsensusStrategy.MAJORITY_VOTE }
            onConsensus   goto "done"
            onNoConsensus goto "write"
        }
        end("done", ExitStatus.SUCCESS)
    }
}
```

---

## Stack

| Module                                                                                    | Role                                                                                              |
|:------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|
| **[hensu-dsl](https://github.com/hensu-project/hensu/tree/main/hensu-dsl)**               | Type-safe Kotlin DSL – compiles `.kt` workflow definitions into portable JSON artifacts           |
| **[hensu-core](https://github.com/hensu-project/hensu/tree/main/hensu-core)**             | Zero-dependency Java engine – state transitions, rubric evaluation, agent orchestration           |
| **[hensu-server](https://github.com/hensu-project/hensu/tree/main/hensu-server)**         | Multi-tenant GraalVM native server – MCP split-pipe transport for secure remote tool execution    |
| **[hensu-cli](https://github.com/hensu-project/hensu/tree/main/hensu-cli)**               | Developer CLI – `run`, `build`, `push`, `pull`, `list`, `attach`; local daemon keeps the JVM warm |

---

The server is a pure orchestrator – it never executes user-supplied code. Tool calls route through
MCP to tenant clients via an outbound SSE connection. No inbound ports, no firewall rules on the
client side. The binary ships as a GraalVM native image.

---

<div align="center">

[Monorepo](https://github.com/hensu-project/hensu) · [DSL Reference](https://github.com/hensu-project/hensu/blob/main/docs/dsl-reference.md) · [Architecture](https://github.com/hensu-project/hensu/blob/main/docs/unified-architecture.md) · [Spring Reference Client](https://github.com/hensu-project/hensu/tree/main/integrations/spring-reference-client)

---

Java 25 · Kotlin · Quarkus · GraalVM Native Image · MCP

<sub>Hensu™ and the axolotl logo are trademarks of Aleksandr Suvorov.<br>
Copyright 2025–2026 Aleksandr Suvorov. All rights reserved.</sub>

</div>
