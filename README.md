# eResource Scheduler

Cursor plugin that connects agents to [eResource Scheduler](https://www.eresourcescheduler.com) through ERS's remote [Model Context Protocol](https://modelcontextprotocol.io/) server.

Manage resources, projects, bookings, timesheets, rates, and reports in the signed-in ERS workspace.

## Install

1. Open **Cursor Settings → Plugins**.
2. Search for **eResource Scheduler**.
3. Click **Install**, then complete the ERS sign-in prompt.

Or run `/add-plugin ers` in chat.

## MCP

```json
{
  "mcpServers": {
    "eRS": {
      "type": "http",
      "url": "https://test.eresourcescheduler.cloud/mcp"
    }
  }
}
```

Auth is OAuth 2.0 against ERS. Cursor prompts for ERS sign-in when the plugin connects.

## Skills

| Skill | What it does |
|---|---|
| `example-skill` | Placeholder that lists what a skill folder can contain. Replace it with real skills. |

## Notes

- Tool calls run as the ERS user who authorizes the connection and cannot exceed that user's permissions.
- This plugin currently points at the ERS test MCP endpoint.

## Docs

- Product: https://www.eresourcescheduler.com
- Server URL: https://test.eresourcescheduler.cloud/mcp

Logo is the official eResource Scheduler product icon, in `assets/logo.png`.

## License

MIT
