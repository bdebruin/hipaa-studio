# HIPAA Studio

Local, air-gapped [HIPAA X12](https://hipaastudio.com/) EDI validation for Windows. The desktop Workbench and AI Agents (MCP) share one validation engine, so interactive and agent-driven checks produce the same findings. Enterprise adds a Windows/Linux CLI and deployment tooling for pipelines.

**This repository is public product and MCP listing documentation only.** It does not contain source code, binaries, or license keys. The MCP server ships as `hipaastudio_mcp.exe` with a HIPAA Studio build.

- Website: [https://hipaastudio.com](https://hipaastudio.com)
- For agents (MCP): [https://hipaastudio.com/agentic/](https://hipaastudio.com/agentic/)
- Getting started: [https://hipaastudio.com/getting-started/](https://hipaastudio.com/getting-started/)
- Documentation: [https://hipaastudio.com/docs/](https://hipaastudio.com/docs/)
- What it checks: [https://hipaastudio.com/what-it-checks/](https://hipaastudio.com/what-it-checks/)
- Trial: [https://hipaastudio.com/trial/](https://hipaastudio.com/trial/)
- Pricing: [https://hipaastudio.com/pricing/](https://hipaastudio.com/pricing/)

## What it does

Files are read from disk and validated in memory — no upload, no account, and no network call required for validation.

HIPAA Studio runs SNIP 1–4 checks on 5010 (the current mandated HIPAA transactions) and also supports 8060 validation, 5010↔8060 conversion, and X12-to-JSON export. Anything the target version needs that the source doesn't hold is flagged for review, not guessed.

Supported 5010 transaction guides include 837P/I/D, 835, 834, 820, 270/271, 276/277, 278, and 275 (005010X210). It also ships 006020X313 (277 request for additional information) and 006020X314 (275 attachments). 8060 validation covers 008060X322 (835), 008060X323 (837P), 008060X324 (837I), 008060X325 (837D), 008060X329 (276/277), 008060X332 (270/271), 008060X333 (834), 008060X334 (820), and 008060X342 (278) only. Confirm with the build's guide list.

Public anonymous download of binaries is closed. Request a free 30-day Professional trial on the [trial page](https://hipaastudio.com/trial/); Support emails the trial license and private download instructions to the work email you provide. MCP is already included with the download.

## AI Agents (MCP)

AI Agents (MCP) runs locally (VS Code, Copilot, Claude Code, Cursor) and uses the same engine as the Workbench and CLI. MCP runs over standard input and output. The agent starts `hipaastudio_mcp.exe` as a child process on the machine the files are on. It listens on no port. Nothing is deployed, and no claim crosses a network because of it.

Humans use the Workbench. Ops and CI use the CLI (Enterprise). Agents call MCP, and MCP uses the CLI under the hood.

### Example client config

The following examples are **illustrative setup only**. Adjust the executable path to the `hipaastudio_mcp.exe` in your licensed build. Paths below are examples, not a discovery of your install.

Claude Code (from the [For agents](https://hipaastudio.com/agentic/) page):

```bash
claude mcp add hipaastudio -- "C:\Program Files\HIPAA Studio\hipaastudio_mcp.exe"
```

Example MCP client config (VS Code / Cursor-style):

```json
{
  "mcpServers": {
    "hipaastudio": {
      "command": "C:\\Program Files\\HIPAA Studio\\hipaastudio_mcp.exe"
    }
  }
}
```

In VS Code chat you can also ask to connect an MCP server and point it at your local `hipaastudio_mcp.exe`. Step-by-step stills are on [Getting started](https://hipaastudio.com/getting-started/).

### Tools

Eleven tools, as documented on [For agents](https://hipaastudio.com/agentic/). Values are omitted by default. `validate_folder` never returns values. `validate_file` and `read_document` return them only when `include_values` is true.

| Tool | What it does |
| --- | --- |
| `validate_file` | One file, findings by rule and location. Values omitted by default. `snip` selects SNIP levels (default all four). Pass `profile` only when a trading-partner rule profile is named; `strict` ignores it and validates to the standard. `include_values: true` adds messages and values. |
| `validate_folder` | Every file under a folder, counts per file and per rule. Never includes values. Same `snip`, `profile`, and `strict`. Stops after `max_files` (default 500). |
| `read_document` | The file as JSON: interchange, group, transaction, and loop. Not a validity check. Values masked as `***` by default; closed code-list qualifiers are kept. `include_values: true` returns the whole claim. |
| `list_guides` | The implementation-guide identifiers this build carries. A guide that is not listed is parsed and not checked. |
| `write_redacted_report` | Validate one file and write a redacted report to `out_path` (`json`, `html`, `html-errors`, or `document`). The tool returns the path and counts, not the report. 999, 824, and TA1 cannot be redacted. |
| `search_rules` | Rule identifier, SNIP level, guide, and message. No file data. |
| `generate_sample` | A minimal synthetic transaction for one guide (required loops and segments once). Fake values; NPIs fail the check digit on purpose. Not a transaction to send. |
| `convert_version` | 5010↔8060 for the 835, the three 837s, and the 278. Omit `out` to report what would be lost without writing; supply `out` to write. 5010 does not carry the demographic detail 8060 requires (DMH); converting back drops DMG03. Output is not necessarily conformant. Test use only, not certified. Anything the target version needs that the source doesn't hold is flagged for review, not guessed. |
| `convert_folder` | The same conversion for a folder. `out` is required. Files that cannot be converted are skipped. Same gaps, and test use only. |
| `deidentify_file` | Replace direct identifiers and shift dates, keeping structure, qualifiers, codes, amounts, and control numbers. Omit `out` to report without writing. The answer is counts, not the file. Not a HIPAA Safe Harbor determination; free text is not read. |
| `deidentify_folder` | The same pass for a folder, one seed across files. Writes a new folder unless `in_place` is set. Not Safe Harbor. |

### Exit codes

Exit codes matter for agents. Especially code 3.

| Exit code | Meaning | What an agent should do |
| --- | --- | --- |
| 0 | Validated, no errors | Pass it on. |
| 1 | Validated, errors found | Read findings and act. |
| 2 | Tool fault | Stop. Not a claim problem. |
| 3 | No errors found, but at least one file was not fully checked. For example, no guide for the transaction in this build (nothing was checked), or a rule that could not be evaluated from the file (partly checked). | Do not treat as a pass. Find out which files were not fully checked, and why. |

### Example prompts

These example prompts are from the public [For agents](https://hipaastudio.com/agentic/) page, for use once `hipaastudio_mcp.exe` is connected:

- "Convert these 1,000 files from 5010 to 8060"
- "Validate them and create a report showing file name and SNIP 1-4 number of errors"
- "Convert the 837P files to JSON"
- "Deidentify these X12 files and write safe copies I can share without PHI"

### Example validation output

**SYNTHETIC EXAMPLE DATA ONLY.** This is not a real claim, not a real patient, not a real provider, and not output from a production run. Default findings omit claim values (rule, SNIP, severity, loop, segment, and element position only), matching the public tool contract.

```json
{
  "note": "SYNTHETIC EXAMPLE DATA ONLY. Not a real claim or a real validation run.",
  "file": "example-837p-synthetic.x12",
  "guide": "005010X222A1",
  "exit_code": 1,
  "summary": {
    "errors": 2,
    "warnings": 1,
    "not_fully_checked": 0,
    "snip_1": 0,
    "snip_2": 1,
    "snip_3": 1,
    "snip_4": 0
  },
  "findings": [
    {
      "rule": "SEGMENT-REPEATS-MORE-TIMES-THAN-ALLOWED",
      "snip": 2,
      "severity": "error",
      "loop": "2400",
      "segment": "SV1",
      "element": null
    },
    {
      "rule": "CLAIM-AMOUNT-DOES-NOT-EQUAL-SUM-OF-SERVICE-LINES",
      "snip": 3,
      "severity": "error",
      "loop": "2300",
      "segment": "CLM",
      "element": "CLM02"
    }
  ]
}
```

The rule names above are taken from HIPAA Studio's public [rule reference](https://hipaastudio.com/rules/) titles, used here only as labels in a made-up example.

## Pricing

Published pricing, per year, as shown on [hipaastudio.com/pricing](https://hipaastudio.com/pricing/):

- **Starter:** $300 per license, for 1 to 4 licenses, open to individuals and companies
- **Team:** flat $1,490 per year for up to 10 users
- **Enterprise:** $4,900 with unlimited users (CLI, TPM, pipelines)
- **Multi-site / TPA:** licenses from $9,900 a year (each covered entity holds its own X12 license)

A **free 30-day trial** is available via request. The trial is Professional: the Workbench and AI Agents (MCP), with SNIP 1–4 on 5010, 8060 validation, conversion both ways, and X12-to-JSON.

Every customer company needs its own X12 license, bought directly from X12 at [https://x12.org/products/licensing-program](https://x12.org/products/licensing-program). One X12 license covers your whole team. A HIPAA Studio license does not include one. A Starter individual buyer is the licensee and needs its own X12 license. Enterprise and Multi-site / TPA license terms: licensing@hipaastudio.com.

Payment and licensing details that are not published on the website are available from licensing@hipaastudio.com.

## Official MCP Registry

A draft `server.draft.json` is in this repository for a future listing under the namespace `io.github.bdebruin/hipaa-studio`.

It is **not published**. The official MCP Registry at [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io) accepts either a public package (npm, PyPI, NuGet, Cargo, OCI, or an MCPB artifact on a GitHub or GitLab release) with namespace verification, or a public remote (Streamable HTTP) URL. HIPAA Studio's MCP server is a local stdio executable bundled with the licensed desktop/Enterprise build. There is no public package and no public remote endpoint, and a remote endpoint would be at odds with the product's local, air-gapped design.

A later official listing would need a public package that points at the local stdio server, for example an MCPB GitHub Release whose `server.json` uses `"registryType": "mcpb"` and `"transport": { "type": "stdio" }`. Domain namespace `com.hipaastudio/...` would additionally need DNS authentication on hipaastudio.com.

## Support

- Support: support@hipaastudio.com
- Licensing, trials, and Multi-site / TPA: licensing@hipaastudio.com

## Notices

HIPAA Studio LLC is a licensed X12 Commercial Use Partner. Every customer company needs its own X12 license, bought directly from X12; one license covers your whole team. A HIPAA Studio license doesn't include one.

HIPAA Studio is an independent software product. It is not affiliated with, endorsed by, or approved by the U.S. Department of Health and Human Services, X12, Washington Publishing Company, or any government agency. Validation results are advisory and are not a certification of compliance. X12 is a trademark of X12 Incorporated. Use of X12 trademarks does not imply affiliation or endorsement.

© 2026 HIPAA Studio. All rights reserved. The product is licensed under the [HIPAA Studio EULA](https://hipaastudio.com/legal/eula/).
