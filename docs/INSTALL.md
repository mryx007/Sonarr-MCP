# Installation on Windows

## 1. Requirements

- **Windows** with **Python 3.10 or newer**.
  [Download Python for Windows](https://www.python.org/downloads/windows/).
- **Sonarr 4**, running locally or reachable on your network.
- **Codex**, or another MCP client that supports local STDIO servers.

Open PowerShell and check Python:

~~~powershell
python --version
~~~

If Python is not found or the Microsoft Store opens, finish installing
Python, make it available on PATH, and open a new PowerShell window.
No additional Python packages or separate OpenAI API key are required.

## 2. Download Sonarr-MCP

**Without Git:** choose **Code → Download ZIP** on the
[repository page](https://github.com/mryx007/Sonarr-MCP), extract the ZIP, and
keep the extracted folder in a permanent location, such as
C:\Tools\Sonarr-MCP. Make sure server.py and openapi.json are in that folder.

**With Git:**

~~~powershell
git clone https://github.com/mryx007/Sonarr-MCP.git
cd Sonarr-MCP
~~~

Do not move the folder after configuring your MCP client unless you also
update the server path in that client.

## 3. Connect to Sonarr

### Default local installation

If Sonarr uses C:\ProgramData\Sonarr\config.xml, no configuration file is
needed. Sonarr-MCP reads its port, URL base and API key from that file.

The Windows account running your MCP client must be able to read it.

### Different local Sonarr folder

Copy settings.example.json to settings.json and set the path to your
Sonarr configuration file:

~~~json
{
  "config_path": "D:\\SonarrData\\config.xml"
}
~~~

settings.json is local and excluded from Git.

### Sonarr on another machine

Set SONARR_URL and SONARR_API_KEY in your MCP client's server environment.
Get the API key from **Sonarr → Settings → General → Security**.

The optional remote example in step 4 shows where to put these values.
Never publish your actual key or configuration file.

## 4. Add the server to Codex

Choose **one** method below.

### Method A: Codex configuration file — no CLI required

Find the full Python executable path:

~~~powershell
python -c "import sys; print(sys.executable)"
~~~

Open your Codex configuration:

~~~powershell
notepad "$env:USERPROFILE\.codex\config.toml"
~~~

Keep the existing contents and add the following block. Replace both example
paths with your actual paths. Single-quoted TOML strings keep Windows
backslashes unchanged.

~~~toml
[mcp_servers.sonarr]
command = 'C:\Path\To\python.exe'
args = ['-u', 'C:\Tools\Sonarr-MCP\server.py']
~~~

If a section named [mcp_servers.sonarr] already exists, update that section
instead of adding a duplicate.

**Remote Sonarr only:** add the following under the same server configuration,
using your real address and API key locally:

~~~toml
[mcp_servers.sonarr.env]
SONARR_URL = "http://192.168.1.10:8989"
SONARR_API_KEY = "YOUR_SONARR_API_KEY"
~~~

This environment section is not needed for the default local installation.

### Method B: Codex CLI

If the codex command is already available, open PowerShell in the folder
containing server.py and run:

~~~powershell
$serverPath = (Resolve-Path ".\server.py").Path
$pythonPath = python -c "import sys; print(sys.executable)"
codex mcp add sonarr -- $pythonPath -u $serverPath
~~~

If codex is not found, use Method A.

### Other MCP clients

Use the full Python executable path as the command and these two arguments:

1. -u
2. The full path to server.py

Select the **STDIO** transport. Configure the same optional environment
variables as above if Sonarr is remote.

## 5. Optional: permanently approve writes

By default, your client's normal approval rules apply. The server does not
change permissions itself.

If you explicitly want Codex to allow all Sonarr writes without another
approval prompt, add this section to your own Codex configuration:

~~~toml
[mcp_servers.sonarr.tools.sonarr_apply_change]
approval_mode = "approve"
~~~

**This also permits deletions and commands through this tool.** It is not
limited to quality profiles. Change previews and encrypted snapshots remain
enabled. Managed policies may still restrict access.

To return to the client's default approval behavior, remove this section.
Restart Codex after either change.

## 6. Restart and verify

Restart Codex, then ask:

> Show my Sonarr quality profiles without changing anything or querying indexers.

A successful response should show the profiles from your Sonarr instance.

You do not need to start server.py manually or leave a PowerShell window
open; the MCP client starts the server when needed.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Python not found | Install Python, check PATH, then reopen PowerShell. |
| Server fails to start | Check the absolute Python and server.py paths. Keep openapi.json beside server.py. |
| No Sonarr API key | Check config_path and file permissions, or supply SONARR_API_KEY in the client's environment. |
| Cannot reach Sonarr | Confirm Sonarr is running and check its host, port and URL base. |
| Duplicate TOML section | Edit the existing sonarr section instead of adding another one. |
| Approval is still required | Restart Codex and verify the per-tool approval section. Managed policies can override local choices. |
| A plan expired or vanished | Prepare a new preview; plans expire after 30 minutes or a server restart. |

Windows is required for encrypted write snapshots. Snapshots do not recover
deleted media or undo external actions. See the
[README](../README.md#important-details) for the remaining limitations.

[Official Codex MCP documentation](https://learn.chatgpt.com/docs/extend/mcp)
