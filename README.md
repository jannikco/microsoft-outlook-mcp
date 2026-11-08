# Microsoft Outlook mailbox context for AI assistants

Collect message subjects and mailbox folders to prepare a user-authorized inbox summary.

[Start locally](#get-started) · [Connect your assistant](#connect-your-assistant) · [Available tools](#available-tools) · [Hosted PipMCP setup](#use-pipmcp-for-production)

## Built for useful workflows

- **Build a Microsoft Outlook brief.** Read a bounded page of delegated mailbox messages; requires Mail.Read. Make that provider response available to your assistant before it proposes a next action.
- **Investigate a chosen Microsoft Outlook result.** Read folders in the delegated mailbox. Use an account or record returned by your provider rather than guessing an identifier.
- **Keep the workflow read-only.** The shipped operations expose Microsoft Outlook reads. Combine their returned context in your assistant without adding a provider write operation.

## Why use this package?

Get a focused Microsoft Outlook MCP integration without assembling request mappings and a stdio transport yourself. The package provides documented input schemas, provider authentication handling and an example CLI, with the same tools available to your assistant.

The selected tools keep this Microsoft Outlook workflow read-only.

## Get started

**Requirements:** Node.js 22 or newer, npm, and access to the provider account you intend to use.

```sh
git clone https://github.com/jannikco/microsoft-outlook-mcp.git
cd microsoft-outlook-mcp
npm ci --ignore-scripts
npm test
npm start
```

`npm start` shows the tool inventory and workflow offline. It does not authenticate or call the provider. The tests use provider fixtures and stdio transport; they do not modify your account.

### Configure your provider account

The current release connects directly to the provider with **your own credentials**. It does not require a PipMCP key for local execution. Configure the environment of the process or MCP client that runs the server; `.env` files are not loaded automatically.

| Environment variable | What to supply |
| --- | --- |
| `MICROSOFT_OUTLOOK_ACCESS_TOKEN` | Your provider OAuth access token; use the documented login helper where supported. |

Register your own Entra application as a native/mobile-and-desktop public client with the loopback URI http://127.0.0.1:42813/callback; HTTP 127.0.0.1 redirects may require editing the application manifest. Set MICROSOFT_OUTLOOK_CLIENT_ID and grant delegated Mail.Read. No client secret is used for this public PKCE flow. Your tenant may require administrator approval. No PipMCP account is needed.

[Provider credential/reference documentation](https://raw.githubusercontent.com/microsoftgraph/msgraph-metadata/master/openapi/v1.0/openapi.yaml). Keep credentials out of commits, issue reports and shared client configuration.

### Optional local OAuth helper

Use your own provider OAuth application. Configure `MICROSOFT_OUTLOOK_CLIENT_ID`, `MICROSOFT_OUTLOOK_OAUTH_SCOPES` as described in connector.json and the provider reference. For refreshes, the server process must also have the required application configuration. A manually supplied access token remains an alternative.

```sh
node local-auth/cli.mjs login ./connector.json
node local-auth/cli.mjs status ./connector.json
```

The helper stores credentials in the native OS credential store where supported. Login does not establish that every tool or permission works. See [local auth documentation](./local-auth/README.md).

### Run your first provider read

After configuring the environment, run `list_messages`:

```sh
npm start -- --call list_messages '{"top":10,"select":"id,subject,receivedDateTime"}'
```

Replace sample account, record, file and project IDs with values from your authorized account. This command makes a real provider request and returns its result for you to inspect.

## Available tools

| Capability | MCP tool | Effect | Required inputs |
| --- | --- | --- | --- |
| Read a bounded page of delegated mailbox messages; requires Mail.Read. | `list_messages` | Read | None |
| Read folders in the delegated mailbox. | `list_mail_folders` | Read | None |

Tool schemas and bindings are in [connector.json](./connector.json). These are the selected operations this package ships.

### A practical tool sequence

1. Use `list_messages` — Read a bounded page of delegated mailbox messages; requires Mail.Read.
2. Use `list_mail_folders` — Read folders in the delegated mailbox.

## Connect your assistant

Add this stdio server to an MCP-compatible client. Replace the absolute path and credential placeholders. Use your client’s secret/environment controls instead of committing this configuration.

```json
{
  "mcpServers": {
    "microsoft-outlook-mcp": {
      "command": "node",
      "args": [
        "/absolute/path/microsoft-outlook-mcp/server.mjs"
      ],
      "env": {
        "MICROSOFT_OUTLOOK_ACCESS_TOKEN": "YOUR_MICROSOFT_OUTLOOK_ACCESS_TOKEN"
      }
    }
  }
}
```

`node server.mjs` starts the MCP server; `npm start` runs the example CLI. The server reserves stdout for MCP messages. If the client does not inherit your shell environment, pass the required variables explicitly.

## Read-only scope

The shipped Microsoft Outlook tools are read-only. This package does not provide a write command or silently expand the provider permissions.

## Use PipMCP for production

The local package is useful when you want to run these selected Microsoft Outlook operations in your own process. PipMCP provides a hosted place to configure provider connections, publish tools, issue endpoint access tokens, apply server-side policy and approvals, and inspect execution/usage records. Provider credentials and hosted execution are configured in that service.

**[Set up Microsoft Outlook in PipMCP](https://pipmcp.com/github/start?app=microsoft-outlook&integration=microsoft-outlook-mcp&utm_source=github&utm_campaign=github-microsoft-outlook-mcp&utm_content=readme)**

A managed catalog template is listed, but that is not evidence that its tools match this repository or that your account is connected. Complete provider setup and inspect the endpoint’s actual tool inventory and permissions before use.

Local credentials are not uploaded by this repository. A hosted endpoint uses its own PipMCP endpoint access token and published tool inventory; the local and hosted credentials are separate.

## Scope and validation

- The installed package has HTTP request fixtures and MCP stdio `tools/call` tests.
- Fixture success checks client behavior; it does not establish provider authentication, paid-plan access or live account execution.
- No live provider execution is claimed for this package by these tests.
- Hosted tool names, schemas, permissions and availability can differ from this local package. A queued or accepted job/message is not proof of completion or delivery. Provider API plans, account rules and limits apply.
- See [integration.json](./integration.json) for operation-scoped evidence and the provider-specific limits above.

## FAQ

**Do I need a Microsoft Outlook account?**

You need the provider account, token and permissions required by these operations. Provider charges and API limits remain with that account.

**Does this version require a PipMCP API key?**

This package uses your provider credentials for local execution. To use the hosted service, configure the provider connection in PipMCP and use the endpoint access token issued there.

**Why is a tool missing or a request rejected?**

This repository exposes the selected tools above. Check the required environment variables, provider scopes, account/region requirements and input IDs first. Hosted endpoints have their own published inventory and may exclude tools by role or policy. Do not retry a failed write without checking its outcome.

**Is this an official provider SDK or a hosted-service guarantee?**

This is an independently maintained MCP integration. Its fixtures and source references do not imply provider endorsement, unrestricted API access, or identical hosted behavior.

## Guides and related workflows

- [Microsoft Outlook integration guide](https://jannikco.github.io/pipmcp-connectors/integrations/microsoft-outlook/)
- [Microsoft Outlook workflow and tool sequence](https://jannikco.github.io/pipmcp-connectors/workflows/microsoft-outlook/)
- [More integration workflows](https://jannikco.github.io/pipmcp-connectors/workflows/)
- [Typed LeadAdapter API client](https://github.com/jannikco/linkedin-typescript-sdk)
- [LeadAdapter Python API client](https://github.com/jannikco/linkedin-python-sdk)
- [LinkedIn API quickstart](https://github.com/jannikco/linkedin-api)
- [LinkedIn people and post search](https://github.com/jannikco/linkedin-search-api)
- [LinkedIn post engagers](https://github.com/jannikco/linkedin-post-engagers)
- [LeadAdapter LinkedIn MCP server](https://github.com/jannikco/linkedin-mcp)
- [OpenAPI to MCP import starter](https://github.com/jannikco/openapi-to-mcp)
- [Swagger to MCP import starter](https://github.com/jannikco/swagger-to-mcp)
- [Microsoft Outlook API and credential reference](https://raw.githubusercontent.com/microsoftgraph/msgraph-metadata/master/openapi/v1.0/openapi.yaml)
- [Additional official source 2](https://learn.microsoft.com/en-us/graph/api/overview)
- [Additional official source 3](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
- [Additional official source 4](https://learn.microsoft.com/en-us/graph/api/user-list-messages?view=graph-rest-1.0)
- [Additional official source 5](https://learn.microsoft.com/en-us/entra/identity-platform/reply-url)

Maintained by [jannikco](https://github.com/jannikco). MIT for this repository’s code; dependencies retain their licenses. [Report a reproducible issue](https://github.com/jannikco/microsoft-outlook-mcp/issues) with redacted input and error details. For credentials or security issues, use [private contact](https://pipmcp.com/contact).
