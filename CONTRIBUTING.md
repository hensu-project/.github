# Contributing to Hensu

First, thank you for your interest in Hensu. We are currently in the **Alpha (Feature-Complete)** phase, focusing on the stabilization and verification of our core engine.

## Our Philosophy
We prioritize architectural integrity and strict statelessness. Contributions should align with the "Describe, don't implement" vision.

## How to Contribute

### 1. Reporting Bugs
Please use the GitHub Issue tracker. Since we are at ~70% core verification, detailed logs and reproducible DSL snippets are highly appreciated.

### 2. Development Environment
To contribute to the codebase, you will need:
* **JDK 25+**
* **GraalVM** (for native image testing)
* **Docker** (for integration testing)

### 3. Testing Standards
Hensu is an infrastructure project; stability is our priority.
* Every PR must include unit or integration tests for the DSL-to-Server execution path.
* We utilize a reactive, non-blocking test model. Please ensure new features do not introduce blocking calls to the Core engine.

### 4. Pull Request Process
1. Fork the repository and create your branch from `main`.
2. Ensure your code adheres to the project's Kotlin/Java formatting standards.
3. Update the documentation if you are modifying DSL syntax.
4. Maintainers will review the architectural impact before merging.

## Code of Conduct
By participating in this project, you agree to abide by our [Code of Conduct](./CODE_OF_CONDUCT.md).
