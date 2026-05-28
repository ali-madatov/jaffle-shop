<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

## dbt Agent Skills

This project uses dbt agent skills from [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills).
The skills are installed as `.instructions.md` files in `.github/instructions/` and are **automatically loaded by VS Code Copilot** based on file context.

### Mandatory Skill Usage

When working in this project, you MUST follow the relevant dbt agent skills:

| Skill | File | When to Use |
|-------|------|-------------|
| **Analytics Engineering** | `dbt-analytics-engineering.instructions.md` | Building/modifying dbt models (`.sql` files) |
| **Unit Testing** | `dbt-unit-testing.instructions.md` | Adding unit tests to model YAML files |
| **Semantic Layer** | `dbt-semantic-layer.instructions.md` | Creating/modifying semantic models, metrics, dimensions |
| **Running Commands** | `dbt-commands.instructions.md` | Executing any dbt CLI command |
| **Answering Questions** | `dbt-answering-questions.instructions.md` | Answering business/data questions |
| **Troubleshooting** | `dbt-troubleshooting.instructions.md` | Diagnosing dbt Cloud job failures |
| **Fetching Docs** | `dbt-fetching-docs.instructions.md` | Looking up dbt documentation |

### Key Rules (Always Apply)

1. **Always use `dbt build`** over `dbt run` — build = run + test
2. **Always use `--select`** — never run the entire project
3. **Always use `--quiet`** with `--warn-error-options '{"error": ["NoNodesForSelectionCriteria"]}'`
4. **Always use `{{ ref() }}` and `{{ source() }}`** — never hardcode table names
5. **Always validate with `dbt show`** before considering a model complete
6. **Never modify a test to make it pass** without understanding why it's failing
7. **Semantic layer first** — when answering data questions, check metrics before writing raw SQL
