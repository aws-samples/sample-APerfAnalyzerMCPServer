## Reporting a Vulnerability

If you discover a potential security issue in this project, we ask that you notify AWS/Amazon Security
via our [vulnerability reporting page](https://aws.amazon.com/security/vulnerability-reporting/) or directly via email to aws-security@amazon.com.
Please do **not** create a public github issue.

## Security Model

This MCP server is a local, single-operator tool. Know what it trusts before you deploy it:

- **It fully trusts its caller.** There is no authentication or authorization layer: any process
  that can talk to it over MCP can call every tool. That is safe because it runs as a stdio
  subprocess of your own MCP client, with your own user's permissions, and nothing else can
  reach it.
- **Never expose it over a network transport** (HTTP, SSE, WebSocket, or a socket bridge), and do
  not connect it to a shared or multi-tenant agent session. It has no defense against callers you
  did not intend.
- **Treat analyzed data as untrusted.** aperf captures and the text in them can steer what your AI
  agent asks the server to do. The server limits what such input can reach: paths stay inside
  `APERF_DATA_DIR`, archives are size-capped and extracted with path-traversal protection, and
  every tool call has a 600-second limit.
- **Process execution is opt-in.** `improve_app` runs a project's test suite only when called with
  `run_tests=true` and only for directories listed in `APERF_APPROVED_PROJECT_DIRS`, using one
  fixed `pytest` command with no shell. Running tests executes that project's own code, so list
  only projects you trust.
- **One outbound network call.** `pricing_status` with `confirm=true` fetches the public AWS EC2
  pricing index over HTTPS. Nothing else connects to the network.
