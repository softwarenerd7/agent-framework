# AGENTS.md

## Cursor Cloud specific instructions

This is a multi-language (Python + .NET) monorepo for the **Microsoft Agent Framework** SDK. It is a library/SDK, not a standalone application.

### Project structure

- **Python** (`python/`): uv workspace monorepo with 9 packages under `packages/`. Uses `uv` for dependency management and `poethepoet` (poe) for task running.
- **.NET** (`dotnet/`): Solution file at `dotnet/agent-framework-dotnet.slnx` targeting .NET 9.0.
- **DevUI frontend** (`python/packages/devui/frontend/`): React/Vite/TypeScript app using yarn.

### Python development

All Python commands run from `/workspace/python/`.

- **Install dependencies**: `uv sync --all-packages --all-extras --dev -U --prerelease=if-necessary-or-explicit --no-group=docs` (or `uv run poe install`)
- **Run all quality checks**: `uv run poe check` (fmt + lint + pyright + mypy + test + markdown-code-lint + samples-code-check)
- **Format**: `uv run poe fmt`
- **Lint**: `uv run poe lint`
- **Type check**: `uv run poe pyright`
- **Tests**: `uv run poe test`
- **Run a specific package's tests**: `uv run poe --directory packages/core test`

See `python/DEV_SETUP.md` for full reference on poe tasks and setup.

**Known issue (as of Feb 2026)**: `opentelemetry-semantic-conventions-ai` v0.4.14 (locked in `uv.lock`) removed `SpanAttributes.LLM_REQUEST_MODEL`, causing import errors in `agent_framework`. When `uv run` is used, it re-syncs to the locked v0.4.14 which breaks imports. **Workaround**: after running `uv sync`, downgrade with `.venv/bin/pip install "opentelemetry-semantic-conventions-ai==0.4.13"`, then run tests with `.venv/bin/python -m pytest` instead of `uv run poe test`. Alternatively, run tests with `uv run` and accept the failure if the upstream lock file hasn't been updated yet.

### .NET development

All .NET commands run from `/workspace/dotnet/`.

- **Build**: `dotnet build agent-framework-dotnet.slnx`
- **Unit tests**: `dotnet test tests/Microsoft.Agents.AI.UnitTests -f net9.0` (and similarly for other `*UnitTests` projects)
- **Format/Lint**: `dotnet format`
- Integration tests require external API keys (OpenAI, Azure); they will fail without configuration. Run unit tests with `-f net9.0` to avoid `net472` targets that require Mono.

### DevUI (hello world without API keys)

The DevUI can be run with mock workflows that don't require API keys:

```bash
cd python/samples/getting_started/devui
/workspace/python/.venv/bin/python -c "
from spam_workflow.workflow import workflow
from agent_framework.devui import serve
serve(entities=[workflow], port=8090, auto_open=False)
"
```

Then open `http://localhost:8090` to see the interactive workflow UI.

### Environment notes

- `uv` is installed at `~/.local/bin/uv`.
- .NET SDK 9.0 is installed at `~/.dotnet/`.
- Both are added to PATH via `~/.bashrc`.
- The DevUI frontend uses yarn (`python/packages/devui/frontend/yarn.lock`).
