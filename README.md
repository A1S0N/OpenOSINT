# OPENOSINT(1) &mdash; General Commands Manual

<div align="center">
  <img src="docs/logo.svg" alt="OpenOSINT" width="320">
</div>

<br>

[![Release](https://img.shields.io/github/v/release/OpenOSINT/OpenOSINT?label=release&style=flat-square)](https://github.com/OpenOSINT/OpenOSINT/releases)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue?style=flat-square)](https://www.python.org/)
[![MCP](https://img.shields.io/badge/protocol-MCP-blueviolet?style=flat-square)](https://modelcontextprotocol.io/)
[![PyPI](https://img.shields.io/pypi/v/openosint?style=flat-square)](https://pypi.org/project/openosint/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

> ⚠️ **Legal Disclaimer**: OpenOSINT is intended for **legal and authorized use only**.
> Users are solely responsible for ensuring their use complies with all applicable laws.
> The authors accept no liability for misuse. See [DISCLAIMER.md](DISCLAIMER.md).

<div align="center">
  <img src="assets/demo.gif" alt="OpenOSINT demo" width="700">
</div>

---

## NAME

**openosint** &mdash; Model Context Protocol server and CLI for Open Source Intelligence.

---

## SYNOPSIS

```
openosint [-v] command [args ...]
openosint email ADDRESS [-t SECONDS]
openosint username HANDLE [-t SECONDS]
```

---

## DESCRIPTION

**openosint** is a modular OSINT framework that exposes 9 intelligence-gathering
tools to large language models via the Anthropic Model Context Protocol (MCP).
It also operates as a conventional command-line interface for direct human
execution.

The framework is built on a non-blocking asynchronous runtime (Python `asyncio`).
All external binaries are invoked as managed subprocesses with hard timeout
enforcement. No LLM is embedded — **openosint** provides the tool surface that
an MCP-compatible client drives autonomously.

---

## ARCHITECTURE

| Layer | Path | Responsibility |
|-------|------|----------------|
| Core tools | `openosint/tools/` | Async wrappers around external OSINT binaries and APIs. No I/O, no UI. |
| MCP server | `openosint/mcp_server.py` | Translates core functions into MCP tool schemas. Routes LLM calls. |
| CLI | `openosint/cli.py` | Human-facing interface. Calls core tools directly. |

No layer imports from a layer above it. The core tools are stateless and have no knowledge of MCP or argparse.

---

## INSTALLATION

Requires Python 3.10 or later.

```bash
git clone https://github.com/OpenOSINT/OpenOSINT.git
cd OpenOSINT
pip install -e .
```

**External dependencies** (must be present in `PATH`):

| Binary | Purpose | Install |
|--------|---------|---------|
| `holehe` | Email account enumeration | `pip install holehe` |
| `sherlock` | Username enumeration (300+ platforms) | `pip install sherlock-project` |
| `sublist3r` | Subdomain enumeration | `pip install sublist3r` |
| `phoneinfoga` | Phone number intelligence | [Download binary](https://github.com/sundowndev/phoneinfoga/releases) |

If a binary is absent, the corresponding tool returns a descriptive error string. The server and CLI remain operational for tools with satisfied dependencies.

**Optional environment variables:**

| Variable | Tool | Purpose |
|----------|------|---------|
| `HIBP_API_KEY` | `search_breach` | HaveIBeenPwned API key — [get one here](https://haveibeenpwned.com/API/Key) |
| `IPINFO_TOKEN` | `search_ip` | ipinfo.io token for higher rate limits |

---

## TOOLS

### search_email

Enumerates online services and social accounts associated with an email address using [holehe](https://github.com/megadose/holehe).

**MCP parameter:** `email` (string, required) — target email address.

**CLI:**
```bash
$ openosint email target@example.com
$ openosint email target@example.com -t 60
```

**Example output:**
```
OSINT results for 'target@example.com':

[+] Spotify        https://open.spotify.com/user/target
[+] WordPress      https://wordpress.com/target
[+] Gravatar       https://gravatar.com/target
[+] Office365      email used
```

---

### search_username

Searches for a username across 300+ platforms using [sherlock](https://github.com/sherlock-project/sherlock).

**MCP parameter:** `username` (string, required) — target username or alias.

**CLI:**
```bash
$ openosint username johndoe99
$ openosint username johndoe99 -t 120
```

**Example output:**
```
OSINT results for username 'johndoe99':

[+] GitHub         https://github.com/johndoe99
[+] Twitter        https://twitter.com/johndoe99
[+] Reddit         https://reddit.com/user/johndoe99
[+] HackerNews     https://news.ycombinator.com/user?id=johndoe99
```

---

### search_breach

Checks if an email address appears in known public data breaches via the [HaveIBeenPwned v3 API](https://haveibeenpwned.com/API/v3).

**Requires:** `HIBP_API_KEY` environment variable set.

**MCP parameter:** `email` (string, required) — target email address.

**Example output:**
```
Found in 2 breach(es) for 'target@example.com':

[+] LinkedIn (2016-05-05) — leaked: Email addresses, Passwords, Names
[+] Adobe (2013-10-04) — leaked: Email addresses, Password hints, Usernames
```

---

### search_whois

Retrieves WHOIS registration data for a domain using [python-whois](https://github.com/richardpenman/whois).

**MCP parameter:** `domain` (string, required) — target domain (e.g. `example.com`).

**Example output:**
```
WHOIS results for 'example.com':

[+] Domain: EXAMPLE.COM
[+] Registrar: ICANN
[+] Created: 1995-08-14
[+] Expires: 2024-08-13
[+] Name Servers: A.IANA-SERVERS.NET, B.IANA-SERVERS.NET
[+] Emails: abuse@iana.org
```

---

### search_ip

Retrieves geolocation, ASN, hostname, and organisation data for an IP address via [ipinfo.io](https://ipinfo.io).

Free tier: 50k requests/month without a token. Set `IPINFO_TOKEN` for higher limits.

**MCP parameter:** `ip` (string, required) — target IP address (e.g. `8.8.8.8`).

**Example output:**
```
IP intelligence for '8.8.8.8':

[+] Ip: 8.8.8.8
[+] Hostname: dns.google
[+] Org: AS15169 Google LLC
[+] City: Mountain View
[+] Region: California
[+] Country: US
[+] Loc: 37.4056,-122.0775
[+] Timezone: America/Los_Angeles
```

---

### search_domain

Enumerates subdomains of a target domain using [sublist3r](https://github.com/aboul3la/Sublist3r).

**MCP parameter:** `domain` (string, required) — target domain (e.g. `example.com`).

**Example output:**
```
Subdomains found for 'example.com':

[+] mail.example.com
[+] dev.example.com
[+] staging.example.com
[+] api.example.com
```

---

### generate_dorks

Generates a set of 12 targeted Google dork URLs for any target string (name, email, username, or domain). No network calls — returns URLs ready to open in a browser.

**MCP parameter:** `target` (string, required) — any target string.

**Example output:**
```
Google dork URLs for 'john.doe@example.com':

[+] "john.doe@example.com"
    https://www.google.com/search?q=%22john.doe%40example.com%22

[+] "john.doe@example.com" site:linkedin.com
    https://www.google.com/search?q=%22john.doe%40example.com%22+site%3Alinkedin.com
...
```

---

### search_paste

Searches Pastebin dumps for mentions of an email address or username via the [psbdmp.ws](https://psbdmp.ws) public API.

**MCP parameter:** `query` (string, required) — email address or username to search for.

**Example output:**
```
Found in 3 paste(s) for 'target@example.com':

[+] https://pastebin.com/aB1cD2eF (2023-04-12)
[+] https://pastebin.com/xY3zA4bC (2022-11-08)
[+] https://pastebin.com/mN5oP6qR (2021-07-30)
```

---

### search_phone

Gathers carrier, country, and line type data for a phone number using [phoneinfoga](https://github.com/sundowndev/phoneinfoga).

**MCP parameter:** `phone` (string, required) — target phone number in E.164 format (e.g. `+14155552671`).

**Requires:** `phoneinfoga` binary in `PATH`. [Download here](https://github.com/sundowndev/phoneinfoga/releases).

**Example output:**
```
Phone intelligence for '+14155552671':

[+] International format: +1 415-555-2671
[+] Country: United States
[+] Carrier: AT&T
[+] Line type: Mobile
```

---

## COMMANDS

```
email ADDRESS [-t SECONDS]
```
Enumerate online services registered against *ADDRESS* using holehe. Default timeout: 120 seconds.

```
username HANDLE [-t SECONDS]
```
Enumerate platforms where *HANDLE* is registered using sherlock. Default timeout: 180 seconds.

**Global flags:**

| Flag | Description |
|------|-------------|
| `-v, --verbose` | Enable debug-level logging to stderr. |
| `-t, --timeout N` | Override default subprocess timeout (seconds). |

---

## CONFIGURATION

### Claude Code

Register the MCP server after installation:

```bash
claude mcp add openosint python /absolute/path/to/OpenOSINT/openosint/mcp_server.py
```

Verify:

```bash
claude mcp list
```

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "openosint": {
      "command": "python",
      "args": ["/absolute/path/to/OpenOSINT/openosint/mcp_server.py"]
    }
  }
}
```

---

## EXAMPLES

Enumerate services registered against an email address:

```bash
$ openosint email target@example.com -t 60
```

Search for a username across all supported platforms:

```bash
$ openosint username johndoe99
```

Enable verbose output:

```bash
$ openosint -v email target@example.com
```

Agentic execution via Claude Code after MCP registration:

```
$ claude
> Investigate target@example.com. If you find an associated username,
  trace it across other platforms and compile a full report.
```

---

## FILES

| Path | Description |
|------|-------------|
| `openosint/mcp_server.py` | MCP server entry point (stdio transport). |
| `openosint/cli.py` | CLI entry point. |
| `openosint/tools/search_email.py` | Email enumeration module. |
| `openosint/tools/search_username.py` | Username enumeration module. |
| `openosint/tools/search_breach.py` | Data breach check module. |
| `openosint/tools/search_whois.py` | WHOIS lookup module. |
| `openosint/tools/search_ip.py` | IP intelligence module. |
| `openosint/tools/search_domain.py` | Subdomain enumeration module. |
| `openosint/tools/generate_dorks.py` | Google dork URL generator. |
| `openosint/tools/search_paste.py` | Pastebin dump search module. |
| `openosint/tools/search_phone.py` | Phone intelligence module. |
| `openosint/tools/exceptions.py` | Shared exception hierarchy. |
| `pyproject.toml` | Project metadata and build configuration (PEP 621). |
| `DISCLAIMER.md` | Legal notice and ethical use policy. |

---

## EXIT STATUS

| Code | Meaning |
|------|---------|
| 0 | Successful execution. |
| 1 | General error (invalid arguments, tool failure). |
| 130 | Terminated by SIGINT (Ctrl-C). |

---

## AUTHORS

Developed by Tommaso Bertocchi.

---

## LICENSE

MIT License. See [LICENSE](LICENSE).

---

*OpenOSINT 2.1.0 &mdash; May 11, 2026*
