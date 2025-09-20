# Context7 Coral Agent
(modern lorem ipsum)
A lightweight, production-friendly agent that watches your Coral workspace for new mentions and replies with answers grounded in your library documentation. It connects to both the Coral MCP server and a local Context7 MCP server to retrieve docs, monitor incoming requests, and post helpful, citation-rich responses.

## What it does
- Listens for new requests (mentions) in Coral.
- Fetches your project or library documentation via the Context7 MCP server.
- Uses an LLM to synthesize answers that quote your docs where possible.
- Responds directly back to the originating Coral thread.
- Runs in a loop with configurable delay and max iterations, suitable for containerized deployment.

## Why it’s useful
- Gives your team instant, documented answers inside Coral.
- Reduces context switching by replying in the same thread where the question was asked.
- Ensures responses are grounded in your own docs, not just generic model knowledge.
- Configurable temperature and token limits for predictable, cost-conscious operation.
- Optional telemetry for observability when running in production.

## How it works (high level)
1. Establishes MCP connections to:
   - Coral (server URL provided via environment) for workspace tools and messaging.
   - Context7 (spawned over stdio) for documentation retrieval tools.
2. Prepares a prompt that instructs the agent to:
   - If needed, fetch the library docs by `context7CompatibleLibraryID`.
   - Repeatedly call `coral.waitForMentions` until there are messages.
   - Analyze the messages, note the request and thread ID.
   - Reply using `coral.sendMessage`, quoting the docs where possible.
3. Runs a repeating prompt loop (configurable delay and max reps) to continuously handle requests.

## Key capabilities
- MCP-native: Uses Coral and Context7 MCP tools end-to-end.
- Grounded answers: Encourages quoting and citing relevant passages.
- Cost controls: Token limits and temperature are user-configurable.
- Telemetry (optional): Can report to OpenAI for operational insights.

## Configuration
Set the following via CLI flags or environment variables. Defaults and behaviors are defined in `agent/src/main.rs`.

Required
- LIBRARY_ID: The Context7-compatible library ID to fetch docs for.

Common settings
- TEMPERATURE: Sampling temperature for the LLM (float).
- MAX_TOKENS: Maximum tokens for each completion (u64).
- ENABLE_TELEMETRY: true/false to enable OpenAI telemetry.
- SYSTEM_PROMPT_SUFFIX: Optional string appended to the system preamble.
- LOOP_PROMPT_SUFFIX: Optional string appended to the loop instructions.
- LOOP_DELAY: Optional human-readable duration (e.g., "5s", "1m") between loop iterations.
- LOOP_MAX_REPS: Maximum number of loop iterations before exit.

Other environment variables
- Coral and OpenRouter credentials and endpoints must be available via environment variables as expected by `coral_rs` and OpenRouter’s client. The agent uses the OpenRouter client with the `GPT_4_1` model by default.

## MCP tools referenced
- Coral tools
  - coral.waitForMentions
  - coral.sendMessage
- Context7 tools
  - get-library-docs (expects `context7CompatibleLibraryID`)

## Deployment
- Container-friendly (see included Dockerfile).
- Starts the Context7 MCP server via stdio: `/app/run.sh`.
- Compatible with Coral MCP protocol version 2024-11-05.

Example (pseudo):
- Build the container.
- Provide environment variables for OpenRouter, Coral, and the agent’s configuration.
- Run the container; it will connect to Coral, fetch docs as needed, and reply to mentions.

## Pricing
Note: Final pricing is maintained in coral-rs and may be filled in or overridden by the publisher. The table below is a placeholder you can edit.

| Plan | What’s included | Token pricing | Monthly fee |
|------|------------------|---------------|-------------|
| Free | • Basic usage • Limited rate • Community support | TBD | $0 |
| Pro  | • Higher limits • Priority responses • Email support | TBD | TBD |
| Team | • Team usage • Usage analytics • SSO (if available) | TBD | TBD |
| Enterprise | • Custom SLAs • Dedicated support • Advanced controls | TBD | Contact us |

Token costs inside the agent loop are managed via ClaimManager in code. Current defaults:
- Input tokens cost: 0.00125 USD per 1,000 tokens (mil_input_token_cost)
- Output tokens cost: 0.01000 USD per 1,000 tokens (mil_output_token_cost)

Update these values to match your actual pricing before publishing.

## Privacy and telemetry
- Telemetry is OFF by default; enable it with ENABLE_TELEMETRY=true.
- When enabled, telemetry uses OpenAI with the selected model for operational insights.
- Responses aim to quote your docs to minimize inadvertent disclosure of unrelated information.

## Limitations
- Requires accessible and up-to-date docs through Context7.
- Depends on OpenRouter and Coral MCP availability and credentials.
- Long-running loops should be monitored in production.

## Support
- Open issues or discussions in your repository.
- Provide contact details or a support email for marketplace users.
