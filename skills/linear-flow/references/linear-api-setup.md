# Linear API Setup

Use this reference when direct Linear GraphQL API access for ordered queue selection is not configured or the user asks how to set it up. The secret must stay outside chat, repository files, Linear issues, and logs.

## Agent Rules

- Prefer `LINEAR_API_KEY` when it is already present in the command environment.
- Otherwise retrieve the key from a local OS secret store inside the same shell/process that performs the API call.
- Use direct API only for ordered queue-selection queries and minimal setup smoke tests such as `viewer`.
- Use the Linear MCP/app for all normal issue reads, descriptions, labels, comments, status changes, and writes.
- Do not run a separate command that prints the secret.
- Do not ask the user to paste the secret into chat.
- Do not store the secret in `planning/linear.md`, `.env` files committed to the repo, skill files, comments, or generated docs.
- If setup is missing, give the user the relevant OS setup command and stop until they confirm it is complete.

## First-Time Setup

Ask the user to create or copy a Linear personal API key from Linear's API settings, then store it locally using one of these OS-specific methods.

### Universal Session Environment

Use this only when a local secret-store CLI is unavailable. The user enters the key into a hidden prompt for the current terminal session; it is not persisted.

POSIX shells:

```bash
read -rsp "Linear API key: " LINEAR_API_KEY
printf "\n"
export LINEAR_API_KEY
```

PowerShell:

```powershell
$env:LINEAR_API_KEY = Read-Host "Linear API key" -MaskInput
```

### macOS Keychain

The user runs this in Terminal and pastes the key only into the hidden prompt:

```bash
read -rsp "Linear API key: " LINEAR_API_KEY
printf "\n"
security add-generic-password -a "$USER" -s codex-linear-api-key -w "$LINEAR_API_KEY" -U
unset LINEAR_API_KEY
```

One-shot API call pattern:

```bash
sh -c 'k="${LINEAR_API_KEY:-$(security find-generic-password -a "$USER" -s codex-linear-api-key -w 2>/dev/null)}"; test -n "$k" || { echo "ERROR: Linear API key is not configured." >&2; exit 1; }; curl -sS https://api.linear.app/graphql -H "Authorization: $k" -H "Content-Type: application/json" --data-binary "$1"; unset k' sh '{"query":"query { viewer { id name } }"}'
```

### Linux Secret Service

Requires a Secret Service provider such as GNOME Keyring and the `secret-tool` CLI.

The user runs this in Terminal and pastes the key only into the hidden prompt:

```bash
read -rsp "Linear API key: " LINEAR_API_KEY
printf "\n"
printf '%s' "$LINEAR_API_KEY" | secret-tool store --label="Codex Linear API key" service codex-linear-api-key account "$USER"
unset LINEAR_API_KEY
```

One-shot API call pattern:

```bash
sh -c 'k="${LINEAR_API_KEY:-$(secret-tool lookup service codex-linear-api-key account "$USER" 2>/dev/null)}"; test -n "$k" || { echo "ERROR: Linear API key is not configured." >&2; exit 1; }; curl -sS https://api.linear.app/graphql -H "Authorization: $k" -H "Content-Type: application/json" --data-binary "$1"; unset k' sh '{"query":"query { viewer { id name } }"}'
```

### Linux pass

Use this when the user already uses `pass`.

The user runs this in Terminal and pastes the key only into the hidden prompt:

```bash
read -rsp "Linear API key: " LINEAR_API_KEY
printf "\n"
printf '%s\n' "$LINEAR_API_KEY" | pass insert -m -f codex/linear-api-key
unset LINEAR_API_KEY
```

One-shot API call pattern:

```bash
sh -c 'k="${LINEAR_API_KEY:-$(pass show codex/linear-api-key 2>/dev/null | head -n 1)}"; test -n "$k" || { echo "ERROR: Linear API key is not configured." >&2; exit 1; }; curl -sS https://api.linear.app/graphql -H "Authorization: $k" -H "Content-Type: application/json" --data-binary "$1"; unset k' sh '{"query":"query { viewer { id name } }"}'
```

### Windows PowerShell SecretManagement

Requires Microsoft's SecretManagement module and a registered vault.

The user runs this in PowerShell and pastes the key only into the secure prompt:

```powershell
Install-Module Microsoft.PowerShell.SecretManagement -Scope CurrentUser
Install-Module Microsoft.PowerShell.SecretStore -Scope CurrentUser
Register-SecretVault -Name CodexSecrets -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault
$key = Read-Host "Linear API key" -AsSecureString
Set-Secret -Name codex-linear-api-key -SecureStringSecret $key
Remove-Variable key
```

One-shot API call pattern:

```powershell
$k = if ($env:LINEAR_API_KEY) { $env:LINEAR_API_KEY } else { Get-Secret -Name codex-linear-api-key -AsPlainText }; if (-not $k) { throw "Linear API key is not configured." }; Invoke-RestMethod -Uri "https://api.linear.app/graphql" -Method Post -Headers @{ Authorization = $k; "Content-Type" = "application/json" } -Body '{"query":"query { viewer { id name } }"}'; Remove-Variable k
```

## Queue Query Use

When `planning/linear.md` contains the needed identifiers, use the path's queue query in a single command by replacing the example JSON body with the canonical query and variables. Keep the key lookup inside that same command.

If the API returns authentication errors, no key, invalid JSON, missing IDs, or an unreliable ordered result, stop and tell the user the ordered queue could not be fetched authoritatively.
