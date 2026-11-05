# Sonarr-MCP

Manage **Sonarr 4** through an MCP client such as Codex. Edit quality profiles
and custom formats, inspect your library, and access the Sonarr API with
change previews and encrypted snapshots.

Runs locally over STDIO using Python's standard library. Requires Windows,
Python 3.10+, and access to Sonarr 4. Sonarr itself may run on another machine.

## Setup

~~~powershell
git clone https://github.com/mryx007/Sonarr-MCP.git
cd Sonarr-MCP

$serverPath = Join-Path (Get-Location) "server.py"
$pythonPath = (Get-Command python -ErrorAction Stop).Source
codex mcp add sonarr -- $pythonPath -u $serverPath
~~~

Restart Codex after registration. Other MCP clients can launch Python with
-u and the absolute path to server.py.

By default, connection details and the API key are read from
C:\ProgramData\Sonarr\config.xml. For a different installation, copy
settings.example.json to settings.json and adjust config_path.
An optional url field overrides the detected address.

Environment overrides: **SONARR_CONFIG_PATH**, **SONARR_URL**,
**SONARR_API_KEY**. Keep credentials and local configuration out of Git.

### Optional Codex write approval

To permanently approve this server's write tool, explicitly add the following
to your own ~/.codex/config.toml, then restart Codex:

~~~toml
[mcp_servers.sonarr.tools.sonarr_apply_change]
approval_mode = "approve"
~~~

This includes **deletions and commands**, not just profile edits. Otherwise,
your client's normal approval rules apply. The server never grants itself
permission. [Codex documentation](https://learn.chatgpt.com/docs/extend/mcp)

## Tools

| Tool | Purpose |
| --- | --- |
| sonarr_overview | Status, profiles, custom formats, and quality definitions |
| sonarr_api_schema | Browse the bundled Sonarr 4 API reference |
| sonarr_read | Call a /api/v3/... GET endpoint |
| sonarr_prepare_profile | Create, clone, or edit a profile or custom format |
| sonarr_prepare_change | Preview a POST, PUT, PATCH, or DELETE |
| sonarr_apply_change | Apply a prepared plan once |
| sonarr_list_backups | List encrypted snapshots |
| sonarr_prepare_restore | Prepare restoration of an updated resource |

## Important details

- Plans expire after 30 minutes or a server restart. Changed snapshot state
  invalidates the plan. Lists in profile edits are replaced in full.
- Snapshots use Windows DPAPI and are tied to the current Windows account.
  Failed snapshot creation blocks the write.
- Snapshots **cannot undo deleted media, downloads, or external actions**.
  Automatic restoration covers individual PUT/PATCH updates only.
  Actions without readable target state snapshot system status instead.
- After a failed write, inspect Sonarr before retrying: it may have succeeded.
- GET requests can have side effects: /api/v3/release may query indexers;
  /api/v3/parse only parses a title.
- Known credential fields and sensitive URL parameters are masked.
  Arbitrary secrets in free text are not guaranteed to be detected.
- No telemetry or OpenAI API calls. Sonarr may contact external services
  depending on the requested operation.
- Responses are limited to 16 MiB. Multipart uploads are not supported.
  The running Sonarr version determines available API features.

## License

[GPL-3.0-only](LICENSE). The unmodified Sonarr OpenAPI reference is distributed
under GPLv3; see [third-party notices](THIRD-PARTY-NOTICES.txt) and
[Sonarr's license](SONARR-LICENSE).

Independent community project, not affiliated with Sonarr or OpenAI.
