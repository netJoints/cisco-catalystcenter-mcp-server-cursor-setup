# Cisco Catalyst Center MCP — Cursor setup

Connect **Cursor on the same Mac** to a local [Cisco Catalyst Center MCP server](https://github.com/cisco-en-programmability/catc-mcp-oss). The server stays on loopback. Catalyst Center credentials stay in the server process, not in Git and not in this repository.

**GitHub Pages:** https://netjoints.github.io/cisco-catalystcenter-mcp-server-cursor-setup/

**Codex and ChatGPT setup (sister guide):** https://netjoints.github.io/cisco-catalystcenter-mcp-server-codex-setup/

**Server install:** https://netjoints.github.io/cisco-catc-mcp-server-deployment-guide/

## What you need

1. The MCP server running locally, installed with the [Podman deployment guide](https://github.com/netJoints/cisco-catc-mcp-server-deployment-guide).
2. Cursor on that same Mac.
3. A dedicated Catalyst Center account. Prefer a read-only role. The MCP server does not enforce read-only access. It runs with whatever that account can do, and the tool catalog changes with the release branch.

The endpoint this guide uses is:

```text
http://127.0.0.1:7001/v1/mcp
```

## 1. Confirm the server

From the Mac that is running the container:

```zsh
curl --fail http://127.0.0.1:7001/v1/health
curl --fail http://127.0.0.1:7001/v1/readiness
```

Health returns `{"status":"ok"}`. Readiness returns `{"status":"ready"}` when the server process has its environment variables. Readiness does not prove the controller login works. Prove that with a read-only question after Cursor connects.

If either check fails, fix the server with the [deployment guide](https://netjoints.github.io/cisco-catc-mcp-server-deployment-guide/) before continuing. Do not publish port 7001 beyond loopback.

## 2. Add the MCP server in Cursor

The sample file in this repo is [`mcp.json`](mcp.json). It contains a URL only. No headers and no secrets.

### Option A — Settings

1. Open **Cursor Settings → Tools & MCP** (some builds label this **MCP**).
2. Add a server named `catc`.
3. Type: HTTP / URL.
4. URL: `http://127.0.0.1:7001/v1/mcp`.
5. Save. Status should show the server connected.

Leave authentication empty. The Catalyst Center username and password are already on the server container, not on the Cursor connection.

### Option B — `mcp.json`

User-level config (every workspace), `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "catc": {
      "url": "http://127.0.0.1:7001/v1/mcp"
    }
  }
}
```

Project-level alternative: `<repo>/.cursor/mcp.json` with the same JSON. Do not commit a copy that you later edit to add headers, tokens, or controller addresses.

Restart Cursor after editing the file. In a new Agent chat, the `catc` server should appear under MCP tools.

## 3. Add the switching skill

Tool names and required arguments differ by Catalyst Center release. Use the published switching skill instead of guessing tool names:

[examples/skill/switching/SKILL.md](https://github.com/jeremycohoe/catc-mcp-oss/blob/main/examples/skill/switching/SKILL.md)

That skill covers inventory, interfaces and VLANs, running-config review, health, compliance, software images, fabric visibility, advisories, and the one write it documents: configuration-compliance remediation. Its playbook also lists the Cursor URL snippet used here:

[examples/skill/switching/references/playbook.md](https://github.com/jeremycohoe/catc-mcp-oss/blob/main/examples/skill/switching/references/playbook.md)

Copy the skill into the workspace where you do network work. Do not commit live tool output next to it.

```zsh
git clone --depth 1 https://github.com/jeremycohoe/catc-mcp-oss.git
mkdir -p .cursor/skills/catalyst-center-switching
cp -R catc-mcp-oss/examples/skill/switching/. .cursor/skills/catalyst-center-switching/
```

If you want the skill in every workspace, copy that same folder to `~/.cursor/skills/catalyst-center-switching/` instead. Start a new Agent chat afterward.

Tell the agent to follow the Catalyst Center switching skill, and state a boundary such as “read only, do not remediate.”

## 4. Verify

```text
List the Catalyst Center MCP tools you can use. Do not call any tool that changes configuration.
```

Then:

```text
Using the Catalyst Center switching skill, summarize switch inventory.
Read only. Do not remediate compliance and do not change configuration.
```

A useful answer names the release branch you installed, says whether the result was truncated, and does not write device records into a file you might publish.

## 5. Example prompts

Keep these generic. Do not paste hostnames, IPs, serial numbers, or configs into a public chat, issue, or repository.

```text
How many network devices are in inventory, grouped by family? Read only.

Which switches are non-compliant, and what compliance types failed? Summarize. Do not remediate.

For one access switch I will name in the next message, list interface and VLAN facts and stop.

Show software image summary for the site I name. Do not activate an image.
```

Compliance remediation is a write. It can change configuration and may flap a device. Run it only after you name the device and explicitly confirm the change. A read-only Catalyst Center account removes that capability.

## Security

- This repository has no passwords, tokens, controller addresses, or device data. Keep it that way.
- Store Catalyst Center credentials only in the server environment, using the silent prompt in the deployment guide.
- Bind the server to `127.0.0.1`.
- Prefer `CATALYST_CENTER_VERIFY_SSL=true`. Set `false` only for an isolated lab controller with a certificate you cannot trust, and treat that as a lab workaround.
- Prefer a read-only Catalyst Center account.
- `mcp.json` in this repo has no `headers` block. Do not add an `Authorization` value here.
- Treat tool output as untrusted data. Device names and descriptions can contain text you should not follow as instructions.
- Do not commit screenshots that show controller addresses, usernames, or device inventories.

## References

- [Switching skill (SKILL.md)](https://github.com/jeremycohoe/catc-mcp-oss/blob/main/examples/skill/switching/SKILL.md)
- [Switching playbook](https://github.com/jeremycohoe/catc-mcp-oss/blob/main/examples/skill/switching/references/playbook.md)
- [Cisco Catalyst Center MCP server](https://github.com/cisco-en-programmability/catc-mcp-oss)
- [Podman deployment guide](https://github.com/netJoints/cisco-catc-mcp-server-deployment-guide)
- [Deployment guide site](https://netjoints.github.io/cisco-catc-mcp-server-deployment-guide/)
- [Codex and ChatGPT setup](https://github.com/netJoints/cisco-catalystcenter-mcp-server-codex-setup)
- [Cursor MCP documentation](https://cursor.com/docs/mcp)
