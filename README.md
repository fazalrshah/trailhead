# 🥾 Trailhead — a field guide to standing up multi-agent OpenClaw systems

OpenClaw is great, but the multi-agent docs are thin and the **gotchas cost real hours.** Trailhead is
the guide we wish we'd had — the specific traps, in the order you hit them, with the fix for each. Born
from building a 14-agent local, air-gapped system end-to-end.

> Not affiliated with OpenClaw — just hard-won field notes. PRs with more gotchas welcome.

---

## The gotchas (each one cost us time)

### Agent / CLI
- **Agent ids are single tokens.** `agents add "Chief of Staff"` fails ("too many arguments"). Use
  `agent01chiefofstaff`. Pick a convention (`agentNNrole`) up front.
- **`openclaw tui` takes the id positionally**, not as a flag: `openclaw tui <id>` ✅, `--agent <id>` ❌.
- **Match the id exactly in every command.** `--agent 12research` ≠ `agent12research`. A near-miss
  silently targets the wrong (or no) agent.
- **Personas via `IDENTITY.md`.** Drop an `IDENTITY.md` in the agent's workspace for deterministic
  persona loading; `agents list` flips from `(config)` to `(IDENTITY.md)` when it's picked up.

### Shell
- **zsh eats multi-line backslash pastes.** A `\`-continued command split across lines often runs the
  first half and errors on the rest (`command not found: --flag`). **Give/accept single-line commands.**
- **`pip install 'pkg[extra]'` must be quoted in zsh** — `[extra]` is a glob, so unquoted it fails with
  "no matches found."

### MCP servers
- **Register the exact path that returns HTTP 200.** OpenClaw mishandles redirects, so a `/mcp` vs `/mcp/`
  mismatch (307 → ignored) breaks the connection. Probe both, register the one that answers 200.
- **Use stateless MCP for host-side servers.** Stateful streamable-HTTP ties sessions to the process;
  restart the server and the gateway dies with `Session not found` (-32600). `mcp.run(..., stateless_http=True)`.
- **Container → host is `host.docker.internal`.** A Dockerized gateway can't reach a host service at
  `localhost` — use `http://host.docker.internal:<port>`. (And the agent's own `exec` runs *in the
  container*, so it can't see host processes — give it a DB/heartbeat-backed status tool instead.)
- **Re-probe after changing a server's tools**, and restart the gateway if the new tools don't show up to
  the agents.

### Architecture
- **The tool-boundary pattern.** Agents should call **MCP tools**, never hold API tokens or hit services
  directly. The agent asks; a deterministic MCP does the privileged thing. Keeps secrets out of prompts
  and makes behavior auditable.
- **Don't add a second orchestrator.** OpenClaw already gives you inter-agent messaging, queues,
  approvals (HITL), and scheduling. Bolting on a separate graph/orchestration layer just doubles the
  failure surface — keep one control plane.
- **Hard guarantees go in code, not prompts.** "Don't bypass the approval gate" in a system prompt is a
  suggestion; enforce real gates in deterministic tools / DB triggers / approvals.

## Related tools (from the same build)

- [Airlock](https://github.com/fazalrshah/airlock) — deterministic egress gate (MCP) for locked-down agents
- [Mosaic](https://github.com/fazalrshah/mosaic) — coverage-verified local RAG indexer
- [Etched](https://github.com/fazalrshah/etched) — append-only audit ledger
- [Turnstile](https://github.com/fazalrshah/turnstile) — concurrency queue for a single local LLM

## Built by

Built by **[KodeKing](https://www.kodeking.net)** · author **[Fazal Shah](https://www.fazalshah.com)**.
We build local, private, multi-agent AI systems. Found another gotcha? Open a PR.

## License

MIT — see [LICENSE](LICENSE).

<!-- fazalshah-personal-site:start -->
## Author and engineering notes

Published by [Fazal Shah](https://fazalshah.com/). Explore my local AI experiments, Mac hardware test notes, agent engineering articles, and public tools on my [personal website](https://fazalshah.com/).

[All public repositories](https://github.com/fazalrshah?tab=repositories) · [YouTube builds and walkthroughs](https://www.youtube.com/@fazalrshah)
<!-- fazalshah-personal-site:end -->
