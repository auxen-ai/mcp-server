# Auxen MCP Server

Auxen's hosted [Model Context Protocol](https://modelcontextprotocol.io/) server. Lets Claude, Cursor, Continue, or any MCP client provision, manage, and tear down private AI model instances on dedicated GPUs — using natural language.

```
Endpoint:        https://api.auxen.ai/mcp
Transport:       Streamable HTTP (stateless)
Auth:            OAuth 2.1 + PKCE-S256 + Dynamic Client Registration (RFC 7591)
Discovery:       /.well-known/oauth-authorization-server (RFC 8414)
                 /.well-known/oauth-protected-resource (RFC 9728)
Spec version:    MCP 2025-06-18
```

## Quickstart

### Add to Claude.ai

1. In Claude.ai → **Settings → Connectors → Add custom connector**
2. URL: `https://api.auxen.ai/mcp`
3. Click **Connect** → walk through the OAuth consent → Approve

That's it. Claude can now call all 10 Auxen tools.

### Add to Cursor / Continue / other MCP clients

In your client's MCP config:

```json
{
  "mcpServers": {
    "auxen": {
      "url": "https://api.auxen.ai/mcp"
    }
  }
}
```

The client will discover OAuth automatically and prompt for browser consent on first use.

## Tools

10 tools across two permission groups (read-only / write-or-destructive). Every destructive tool requires explicit user approval through the client's standard tool-call UX.

### Read-only

| Tool                          | What it does                                                            |
|-------------------------------|-------------------------------------------------------------------------|
| `auxen_list_models`           | List available AI models with size, parameter count, and PAYG rate     |
| `auxen_get_instance_status`   | Get an instance's status, endpoint URL, API key, hourly burn          |
| `auxen_list_instances`        | List all instances on the account (active and historical)              |
| `auxen_get_balance`           | USD credit balance, active instance count, current hourly burn         |
| `auxen_get_schedule`          | Read an instance's schedule (always_on, as_needed, scheduled_window)   |

### Write / destructive

| Tool                          | What it does                                                            |
|-------------------------------|-------------------------------------------------------------------------|
| `auxen_provision_model`       | Spin up a new private model instance. Draws from PAYG balance.         |
| `auxen_destroy_instance`      | Tear down an instance, invalidate endpoint and API key. Irreversible.  |
| `auxen_pause_instance`        | Pause a running instance — compute meter stops; minimal idle charges.  |
| `auxen_wake_instance`         | Wake a paused instance. ~3-minute provisioning window.                 |
| `auxen_set_schedule`          | Configure when an instance is allowed to run.                          |

## What is Auxen?

[Auxen](https://auxen.ai) hosts private AI model endpoints on dedicated GPUs. Pay by the minute, no subscriptions, OpenAI-compatible API.

```
SIZE     MODELS                              RATE (1× capacity)
─────────────────────────────────────────────────────────────────
Small    gemma2-2b, mistral-7b, llama3.2-3b, phi3-mini    $0.10/hr
Medium   llama3.1-8b, qwen2.5-14b, mistral-nemo-12b, …    $0.20/hr
Large    qwen2.5-32b, mistral-small-24b, command-r, …     $0.65/hr
XL       llama3.1-70b, qwen2.5-72b, mixtral-8x22b          $1.50/hr
```

Each Auxen instance is fully private — no shared inference, no per-token fees, no multi-tenant routing.

## Verify the server is live

```bash
$ curl -i https://api.auxen.ai/.well-known/oauth-protected-resource
HTTP/1.1 200 OK
content-type: application/json

{
  "resource": "https://api.auxen.ai/mcp",
  "authorization_servers": ["https://api.auxen.ai"],
  "scopes_supported": ["mcp"],
  "bearer_methods_supported": ["header"]
}

$ curl -i -X POST https://api.auxen.ai/mcp -H 'Content-Type: application/json' -d '{}'
HTTP/1.1 401 Unauthorized
www-authenticate: Bearer realm="https://api.auxen.ai/mcp",
                  resource_metadata="https://api.auxen.ai/.well-known/oauth-protected-resource/mcp"
```

## Related

- [Auxen homepage](https://auxen.ai)
- [Auxen docs](https://auxen.ai/docs)
- [Auxen Python SDK](https://github.com/auxen-ai/auxen-python) — `pip install auxen`
- [Auxen AI SDK Provider](https://github.com/auxen-ai/ai-sdk-provider) — `pnpm add @auxen-ai/ai-sdk-provider`
- [MCP specification](https://modelcontextprotocol.io)

## License

Apache-2.0
